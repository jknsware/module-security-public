**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/os-hardening/_docs/Helpful Email.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

*The following email was sent by [Thomas H Jones II](https://github.com/ferricoxide) in response to an email sent by Josh Padnick.*

Sorry: Gmail saw fit to dump this in my "Junk" folder. Was looking at my Junk folder because a former work colleague had asked me if I'd had a chance to read the message he'd sent me.

On 02/14/2017 18:50, Josh Padnick wrote:
> Hi Thomas,

> I found your name from this blog post, where it looks like you were trying to do something pretty similar to what I'm currently working on. I'm writing to see if you'd be interesting in helping me (on a paid basis) to complete a deliverable for a client.

> **Background**

> My objective is to use Packer to create an Amazon Machine Image (AMI) with several different paths mounted to different filesystems to improve security, exactly as described in your article. What's different is that:
I'm using Packer's amazon-chroot builder to automatically mount an EBS Volume to an existing EC2 Instance, execute a bunch of commands either from the "host" system or from within the chroot'd environment, unmount the volume, and then make an AMI.

> I'm using Amazon Linux instead of something from the RHEL family.
Note that this work is inspired by a client who saw this presentation (slides). 

> **Current Status**

> I'm a senior software engineer, very comfortable with AWS, and usually comfortable with Linux, but the boot process is somewhat new territory for me, and I'm feeling stuck right now. Hence, my reach out.

On the plus side, AWS uses legacy-init rather than systemd, so, a bit less to try to get your head around.

> My specific issue is that after the partition operations I do on the EBS Volume (which used GPT), I can't seem to get the EC2 Instance to launch from my AMI. Using Amazon's relatively new "get instance screenshot", this is what I see:

> ![image](https://cloud.githubusercontent.com/assets/4295964/23933796/2edec100-08fe-11e7-8df4-df9f46cea533.png)

> From my research, I'm guessing that my EC2 Instance can't even read the boot loader from the EBS Volume

Yeah, that's a typical error for there being no boot-block/boot-loader
and therefore doesn't even make it to mounting the kernel. When I modify the EBS Volume right now, I use these commands:

```
sgdisk --delete 1 /dev/xvdf
sgdisk --new 1:0:+50M /dev/xvdf
sgdisk --new 2:0:+4000M /dev/xvdf
...
/sbin/mkfs.ext4 -F -m0 -O ^64bit /dev/xvdf1
/sbin/mkfs.ext4 -F -m0 -O ^64bit /dev/xvdf2
...
```

You'll want to note that, when you move to a multi-partition format such as the above, you sacrifice some of your configurational complexity. While the standard modules within the initramfs have a basic idea what to do if there's more space than is allocated to disk partitions, that "basic idea" is pretty limited. In short, it only knows how to take any "extra" space and grow the last partition. So, bear that in mind.

I opted to do LVM2 for my root filesystems both because of organizational standards and because, with a small hack to the scripts within the initramfs, the entire partition containing my root VG and LVs is grown. This means that if my launched image is 20GiB (or so) larger than the EBS in my AMI, I can carve up that extra 20GiB any way I see fit.

> I eventually mount all my new volumes to /mnt/ebs-volume and load in a tar of the original EBS volume, but overlaid on the new file system structure. But something seems to be wrong with reading the boot loader.

While it's possible/likely that I've resorted to overkill with mine, ensuring that your boot-partition has a valid boot-loader embedded doesn't hurt. You don't need to set a bootable flag on the partition, but my EL6 scripts still do that to.

At any rate, for the boot-sequence to know what it is it's supposed to do and where to find its elements, you need to ensure that your GRUB, your initramfs and your fstab have all the correct device mappings enumerated. In my AMIgen6 project, the HVMprep.sh script takes care of the GRUB pieces — ensuring that the requisite boot blocks have been laid down. The MkCfgFiles.sh scripts (among other things) makes sure an appropriate /boot/grub/grub.conf file exists. My GrowSetup.sh ensures that dracut is run (mostly for my LVM-specific needs, but may be appropos for your requirements as well).

Because you're taring contents from one disk to another rather than doing a chroot-ed install, you may be running into further issues with device IDs - within your grub.conf, your /etc/fstab and your kernel (if you want an illustration of what I'm talking about, use the `blkid` command to show you the UUIDs of your two disks and partitions). Not sure about Amazon, but EL6 and EL7 default to doing things by device UUID. To make things flexible, I changed to using filesystem LABELs, instead. Whatever method you choose, you need to ensure a consistent, flexible device-reference method and that those references are used everywhere devices need to be managed. Typically, this means:

* Create your new disk's partitons
* Select your filesystems' logical name/Labeling option when creating your filesystem
* Ensure that you're able to mount the partitions by label rather than device-path or UUID
* Update your chroot's /etc/fstab file to reference these same labels
* Update your chroot's /boot/grub/grub.cfg to reference these same labels (if using LVM to manage your volumes, ensure that the relevant MD loads and declares are made in this file, as well)
* Use dracut to update your initramfs - particularly since the Amazon Linux AMI's initramfs you tared over is unlikely to have them.

Again, sorry for the tardiness of the reply. Hopefully you were either able to get things sorted or this will arrive to you in time to still be of some use.

-tom

Also, be aware, that if you use packer's amazon-chroot method to create your AMI, you need to ensure that the process you implement to register an AMI from that disk enables SRIOV support. If it does not, instances launched from your custom AMI won't support 10Gbps networking available in M4-generation instance-types.

When we integrated my AMIgenX scripts with Packer, we opted to use the standard ebs-builder and re-use the boot disk as the chroot-target. This meant that, when the AMI was registered, the new AMI inherited the starting-AMI's SRIOV-support and billing-code attributes (former is useful for all AMIs; the latter is mostly of value for license-included AMI's like Red Hat's).

-tom
