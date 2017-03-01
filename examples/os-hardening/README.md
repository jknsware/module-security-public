**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/os-hardening/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# OS Hardening Packer Build Example

This folder contains an example of how to create a hardened OS using the [os-hardening](/modules/os-hardening) Gruntwork 
Script Module. See the module for additional details.
   
Note that our Packer build file uses the [amazon-chroot Packer builder](https://www.packer.io/docs/builders/amazon-chroot.html)
because we wish to create new disk partitions. The module docs explain [how the amazon-chroot builder works](
/modules/os-hardening/#the-packer-amazon-chroot-builder).

## How do you run this example?

To run this example, you need to do the following:

1. If you are running this locally (versus in a CI environment), create a `terraform.tfvars` in this directory that contains
   values for all the required variables in `terraform/vars.tf`. Place this file in *this* directory, not the `./terraform`
    directory.
1. Run `./packer-build.sh`
 
The `packer-build.sh` script will:

- Create an EC2 Instance
- Install Packer
- Upload the files from `./packer`
- Run the packer build to generate the AMI
- Delete the EC2 Instance
- Return the new AMI ID created
