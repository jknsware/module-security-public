**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/os-hardening/partition-scripts/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# OS Hardening Partition Scripts Module

This Gruntwork Scripts Module contains bash scripts that enable you to create multiple disk partitions on your root disk
volume, which will allow you to mount multiple file systems, each with their own mounting options. To learn more about 
disk partitions, see [Why Multiple Disk Partitions Matter](#why-multiple-disk-partitions-matter) below.

## What's Included

The following bash scripts are included:

- **partition-volumes.sh:** This script creates the desired disk partitions, and copies all files from the source AMI's  
  Snapshot to the newly formatted EBS Volume.
- **cleanup.sh:** This script unmounts our new partitions and mounts a single "fake" partition. This is important because
  Packer expects to unmount a single partition to close out its build.

## Quick Start

The easiest way to get all scripts in this Module onto your servers is to use the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```
gruntwork-install --module-name 'os-hardening/partition-scripts' --repo 'https://github.com/gruntwork-io/module-security' --tag 'v0.1.4'

```

Once the scripts are on the server, run the commands directly at the appropriate place in the Packer build:

```
partition-volume \
  --packer-mount-path '/mnt/ebs-volume' \
  --partition '/:4G' \
  --partition '/home:4G' \
  --partition '/tmp:*'
  --partition '/var:4G' \
  --partition '/var/log:4G' \
  --partition '/var/log/audit:4G'
  
cleanup-volume \
  --packer-mount-path '/mnt/ebs-volume' \
  --mount-point '/'
  --mount-point '/home' \
  --mount-point '/tmp' \
  --mount-point '/var' \
  --mount-point '/var/log' \
  --mount-point '/var/log/audit' 
```

Check out the [OS Hardening Example](/examples/os-hardening) to see a fully working Packer build.

For additional documentation on the script arguments, view the source for the individual scripts.

## Background

### Why Multiple Disk Partitions Matter

A conventional Linux setup has a single disk partition for the root volume. In the example below all paths are mounted 
to partition #1:

```bash
$ sudo sgdisk -p /dev/xvda
Disk /dev/xvda: 16777216 sectors, 8.0 GiB
...
Number  Start (sector)    End (sector)  Size       Code  Name
   1            4096        16777182   8.0 GiB     8300  Linux
 128            2048            4095   1024.0 KiB  EF02  BIOS Boot Partition
 
$ mount
...
/dev/xvda1 on / type ext4 (rw,noatime,data=ordered)
...
```

But security-conscious Linux administrators have two major problems with this:

1. We may want certain paths such as `/tmp` to be mounted with different permissions. E.g. never allow a file in `/tmp`
   to be executed.
2. If a runaway program keeps writing data, we want to contain the damage. E.g. a program that keeps dumping data to 
   `/tmp` shouldn't consume all available space used for `/var/log`, too.
    
For this reason, teams sometimes prefer to create multiple disk partitions. In the example, below, this Linux server is
configured with multiple partitions, each with their own mounting options

```bash
$ sudo sgdisk -p /dev/xvda
Disk /dev/xvda: 62914560 sectors, 30.0 GiB
...
Number  Start (sector)    End (sector)  Size       Code  Name
   1            4096         8392703   4.0 GiB     8300  Linux
   2         8392704        16781311   4.0 GiB     8300  Linux
   3        41947136        62914526   10.0 GiB    8300  Linux
   4        16781312        25169919   4.0 GiB     8300  Linux
   5        25169920        33558527   4.0 GiB     8300  Linux
   6        33558528        41947135   4.0 GiB     8300  Linux
 128            2048            4095   1024.0 KiB  EF02  BIOS Boot Partition
 
 $ mount
 ...
 /dev/xvda1 on / type ext4 (rw,relatime,data=ordered)
 /dev/xvda2 on /home type ext4 (rw,nodev,relatime,data=ordered)
 /dev/xvda3 on /tmp type ext4 (rw,nosuid,nodev,noexec,relatime,data=ordered)
 /dev/xvda4 on /var type ext4 (rw,relatime,data=ordered)
 /dev/xvda5 on /var/log type ext4 (rw,relatime,data=ordered)
 /dev/xvda6 on /var/log/audit type ext4 (rw,relatime,data=ordered)
```

Now you can see we have 6 distinct partitions, each 4GB except one which is 10GB. You may also notice that the `/tmp`
file system is mounted with the `noexec` option, which means that no file executions are permitted.

