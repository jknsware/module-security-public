**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/os-hardening/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# OS Hardening

This is a Gruntwork Script Module meant to be used with [Packer](http://packer.io) to build an AMI of a hardened Linux 
OS. At present, the only supported Linux distribution is Amazon Linux. If you wish to add another distribution, please 
contact support@gruntwork.io.

Our OS hardening is based primarily on the guidelines described in in the [Center for Internet Security Benchmarks](
https://benchmarks.cisecurity.org/downloads/benchmarks/), which are freely downloadable. For Amazon Linux, we used the 
**CIS Amazon Linux Benchmark, v2.0.0 06-02-2016**. 

Note that we have not yet implemented the entire CIS Benchmark. At present, the primary implemented OS hardening feature
is mounting multiple partitions. We hope to implement more CIS recommendations over time. 

## Main Components of this Module

There are two major components to this module:

- [ami-builder](ami-builder): This is a Terraform template that launches an EC2 Instance with Packer pre-installed.
- [packer-templates](packer-templates): This is a Packer build template, plus supporting files that will build an AMI 
  from which you can launch a hardened OS.
   
Fundamentally, to generate an AMI you must:

1. Launch the ami-builder EC2 Instance.
2. SSH into the ami-builder EC2 Instance and run `packer build amazon-linux.json` to build the AMI.
3. Terminate the ami-builder EC2 Instance.

### Why do I need to launch a separate EC2 Instance to run Packer? 

This is because we're using the Packer [amazon-chroot](https://www.packer.io/docs/builders/amazon-chroot.html) builder.
See below for additional details on what this is and how to use it.

## How to Use this Module

### How to Set Your EBS Volume Size and Configure The File System Partitions You Want

Before building the AMI with Packer, you may wish to customize the particular file systems and partitions that your 
hardened OS will use. Follow these steps:

1. Set the value of the `ebs_volume_size` variable in the Packer Template (e.g. `amazon-linux.json`) to the desired EBS
   Volume size (in GB).

1. Edit the Packer Template so that the following scripts specify the desired partition paths
   and sizes:
   - `partition-volumes.sh`: For each desired partition, add an argument like `--partition '/home:4G'`. For additional 
      details see [partition-volume.sh](scripts/partition-volume.sh). Note that for the last `--partition` entry only, 
      you may specify `*` for the size to tell the script to create the largest possible partition based on remaining 
      disk space. Also, make sure your partition sizes don't exceed the space available on your EBS Volume!
   - `cleanup.sh`: For each desired partition, add an argument like `--mount-point '/home'`. For additional details see
      [cleanup.sh](scripts/cleanup.sh)
   
   Note that you will redundantly pass the same list of partition paths to each of the above scripts, but only 
   `partition-volumes.sh` needs both the mount point *and* the desired partition size.

2. Edit the `/files/etc/fstab` file to match the partitions from the previous step. Specify any mount options as desired.

That's it! The Packer template will take care of the rest.

### How to Build the AMI with Packer

Now we're ready to build the actual AMI.

1. Launch the [ami-builder](ami-builder) EC2 Instance. We will execute Packer from this EC2 Instance.

2. On your local machine run `rsync` so that your local directory is continually synced to the ami-builder:

   ```
   while true; do rsync -azv ./packer-templates/ ec2-user@54.212.196.166:/home/ec2-user/ami-builder; sleep 1; done
   ```
   
   Be sure to replace the IP address above with the IP address of your EC2 Instance.
   
3. From the ami-builder EC2 Instance, run Packer:
 
    ```
    cd /home/ec2-user/ami-builder
    sudo su
    packer build amazon-linux.json
    ```
    
    Note that it is important to actually use `sudo su` versus just `sudo` since the $PATH variables are different for 
    the `root` user versus the `ec2-user` user. 
    
    The AMI will now build. 

4. When finished, terminate the ami-builder EC2 Instance.

### How to Launch an AMI with an Encrypted EBS Volume

Packer doesn't yet natively support this, but [Pull Request #4584](https://github.com/mitchellh/packer/pull/4584) is 
implementing amazon-chroot support for the `encrypt_boot` property. This will allow us to build an AMI as above, except
that once the AMI is ready, Packer will automatically copy the unencrypted [Snapshot](
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) as an encrypted Snapshot, create a new AMI based
 on the encrypted Snapshot, and delete the original AMI and its underlying unecncrypted Snapshot.

In the meantime, consider running an instance with the root volume unencrypted, but with additional volumes mounted as
encrypted volumes.
    
## How this Module Works

Much of the design for this module is motivated by our need to support multiple [partitions](http://www.tldp.org/LDP/intro-linux/html/sect_03_01.html)
on a single volume, an OS hardening best practice.

### The Packer amazon-ebs builder

Almost all Packer builds for AWS use the [amazon-ebs](https://www.packer.io/docs/builders/amazon-ebs.html) builder. Packer
templates that use this builder work as follows:

1. Packer launches a brand new EC2 Instance and waits to SSH in.
2. Packer executes a series of provisioners (e.g. upload files, run shell scripts).
3. When finished, Packer shuts down the EC2 instance.
4. Packer takes a snapshot.
5. When the snapshot is complete, Packer creates an AMI and terminates the EC2 Instance. 

The primary advantage of the amazon-ebs Packer builder is simplicity and isolation. Each Packer build launches a completely
 fresh EC2 Instance. The primary downsides are (1) Packer builds are slow because you have to launch a completely new 
 EC2 instance for each build, and (2) it is not possible to make any file system changes to the root partition because 
 you can't unmount the very partition on which the OS itself is running. 
 
### The Packer amazon-chroot builder

To address the shortcomings of the amazon-ebs builder, Packer also has the [amazon-chroot](https://www.packer.io/docs/builders/amazon-chroot.html)
builder. Packer templates that use this builder work as follows:
 
1. Out-of-band from Packer, someone launches an EC2 Instance (the "host system").
2. From the host system, run `packer build <packer-template-file>`. 
3. Packer creates an EBS Volume based on the Snapshot that's part of the source AMI specified in the Packer template, 
   attaches it to the host system, and mounts it.
4. Packer executes the `chroot` command against the path where the EBS Volume has been mounted, which starts a process
   that now sees `/path/to/ebs-volume` as the `/` directory.
5. Packer executes any provisioners in the Packer template. Because most provisioners are running in the `chroot`
   environment, they will execute directly on the EBS Volume. 
6. When all provisioners are complete, Packer detaches the volume, takes a snapshot, and creates an AMI from the snapshot.

#### The Packer Remote Shell Provisioner

Most Packer builds use the [remote-shell](https://www.packer.io/docs/provisioners/shell.html) provisioner to apply
configuration. It's called the remote shell provisioner because it executes shell commands on the "remote" EC2 Instance.
When using the amazon-ebs builder, that means the newly launched EC2 Instance runs the shell commands. When using the
amazon-chroot builder, that means the commands are run from within the `chroot` environment.

#### The Packer Local Shell Provisioner

The Packer [local-shell](https://www.packer.io/docs/provisioners/shell-local.html) provisioner executes shell commands
 from your "local" machine. When using the amazon-ebs builder, that means shell commands run from your local machine.
When using the amazon-chroot builder, that means the commands are run from the EC2 Instance where you launched Packer.

This is significant because it means we can execute Packer commands from the host system against the attached EBS Volume.
This allows us to, for example, delete partitions from the attached EBS Volume and add new partitions. It is the only way
 to use Packer to create an AMI with multiple partitions.

## Module Design Decisions

This section discusses some design decisions made when creating this module.

### Which Tool We Used to Partition the Disks

Our choice of tooling was influenced in large part by whether EBS Volumes were originally partitioned to support the
legacy "MBR" format, or the newer GUID Partition Table (GPT) format.

The partition format of a block device refers to how the first few blocks on the disk are structured and where the boot
 loader code resides. Historically, the MBR format has been in wide use, but its primary limitation is that it does not
permit a disk size greater than 2 TB. For this reason, a newer partition table format, GPT, was invented that supports
up to 9.4 Zetabytes! Because Amazon EBS Volumes sometimes need to exceed 2 TB, the GPT format was a natural choice and
is what all modern AMIs use for their underlying EBS Volumes.

But it turns out that not all Linux disk partition management tools support the GPT format. For example, the venerable
`fdisk` has only "experimental" support for GPT. For that reason, we opted to use the newer `gdisk`.

But `gdisk` works with an interactive prompt, whereas we want to execute full commands as part of our Packer build. For
this reason, we use `sgdisk`, which implements all the same functionality of `gdisk` but as a pure command-line tool 
and not an interactive tool.

### Managing Partitions with `gdisk` vs. LVM

Initially, we experimented with using the LVM (Logical Volume Manager) tool to manage our partitions. LVM is a software
tool that applies a unique file system format to a large disk partition and then allows software-based management of LVM
partitions. The primary benefit to using a layer like LVM is that you can dynamically resize disk partitions (including
root!) without taking your disk offline.

This approach was originally inspired by the presentation [OS Hardening and Packer](https://www.youtube.com/watch?v=8h_Y-L1Q8xI&t=1165s),
 where the speaker makes extensive use of LVM with RHEL and the Packer amazon-chroot builder.
 
But LVM requires that the `/boot` directory be mounted to a separate file system, which in turn requires that `/` and
`/boot` be on separate disk partitions. Since the default boot loader configuration for the Amazon Linux EBS Volume 
expects both `/` and `/boot` on a single partition (partition #1, to be exact), separating these two directories meant 
that we needed to reload the boot loader.

But this proved unworkable. Amazon Linux uses an older version of GRUB for its boot loader, and GRUB was unable to find
the "BIOS partition" of the disk where it should write part of the boot loader configuration. After many attempts, we
 eventually gave up on this direction and instead settled for using traditional "on disk" volumes formatted with `gdisk`.
 
It's likely that these issues are specific to Amazon Linux. Also, the only practical consequence of this is that disk
partitions on EBS Volumes cannot be dynamically resized without stopping the instance and separately mounting the EBS 
Volume. But since we anticipate most users will use an immutable infrastructure anyway, we felt that asking users to
who needed additional space to build a new AMI was not unreasonable.

## Gotchas

- Per the []Packer docs for the amazon-chroot builder](https://www.packer.io/docs/builders/amazon-chroot.html), your 
  provisioning scripts must not leave any processes running or Packer will be unable to unmount the filesystem. 

## TODO

- Allow Packer build to be initiated with `terraform apply` using a local provisioner
- Enable encryption on the EBS Volume.
- Implement automated testing.
- Consider setting up new CloudWatch metrics to report available space on multiple disk partitions, not just the root 
disk partition.