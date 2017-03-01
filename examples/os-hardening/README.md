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

1. Build an AMI using Packer
1. Deploy the AMI using Terraform

These steps are described in detail next.

# TODO: Update this section and below on latest way to setup Packer build.

### Build an AMI using Packer 

The code that runs the EC2 instance in this example is an [Amazon Machine Image
(AMI)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) that has been defined in a [Packer
template](https://www.packer.io/) under `packer/amazon-linux.json`. To build an AMI from this template:

1. Install [Packer](https://www.packer.io/).
1. Set up your [AWS credentials as environment variables](https://www.packer.io/docs/builders/amazon.html).
1. Run `packer build amazon-linux.json` to create the AMI in your AWS account. Note down the ID of this new AMI.

### Deploy the AMI using Terraform

Now that you have an AMI, use Terraform to deploy it:

1. Install [Terraform](https://www.terraform.io/).
1. Open up `vars.tf` and set secrets at the top of the file as environment variables and fill in any other variables in
   the file that don't have defaults. This includes the `ami` variable which you should fill in with the ID of the
   AMI you just built with Packer.
1. `terraform get`.
1. `terraform plan`.
1. If the plan looks good, run `terraform apply`.

When the templates are applied, Terraform will output the IP address of the EC2 instance. If you SSH to the instance,
it should have a `/data` folder. This is the EBS volume. It should persist whatever data you put into it in between
restarts of this EC2 instance.