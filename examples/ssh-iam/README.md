**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/ssh-iam/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# SSH IAM Example

This is an example of how to install [ssh-iam](/modules/ssh-iam) on an EC2 Instance so that you can manage SSH access
for the Instance with AWS [Identity and Access Management (IAM)](https://aws.amazon.com/iam/). You can upload your
public SSH Key to your IAM user account and `ssh-iam` will allow you to SSH to the EC2 Instances using your IAM user
name and SSH key for authentication.

**Note**: To make it possible to run automated tests against this example, we build the `ssh-iam` binary locally and
use the Packer `file` provisioner to copy it into our AMI. This is NOT how you would do it in a real-world use case.
Instead, we recommend using the [Gruntwork installer](https://github.com/gruntwork-io/gruntwork-installer):

```json
{
  "type": "shell",
  "inline": "curl -Ls https://raw.githubusercontent.com/gruntwork-io/gruntwork-installer/master/bootstrap-gruntwork-installer.sh | bash /dev/stdin --version {{user `gruntwork_installer_version`}}"
},
{
  "type": "shell",
  "inline": [
    "gruntwork-install --binary-name ssh-iam --tag v0.0.3 --repo https://github.com/gruntwork-io/module-security",
    "sudo /usr/local/bin/ssh-iam install --iam-group MyIamGroup --iam-group-sudo MyIamSudoGroup"
  ],
  "environment_vars": [
    "GITHUB_OAUTH_TOKEN={{user `github_auth_token`}}"
  ]
}
```

## Quick start

To try these templates out you must have Terraform and Packer installed:

1. Upload your public SSH Key to IAM as documented in the [ssh-iam README](/modules/ssh-iam).
1. Create two IAM groups: one with IAM users that need SSH access with sudo privileges and one with IAM users that need
   SSH access without sudo privileges.
1. Build the `ssh-iam` binary: `./packer/build-binary.sh`
1. Build the Packer template under `packer/ssh-iam.json`, passing the name of the groups you created in the previous
   step using the `iam_group_sudo` and `iam_group` variables:
   `packer build -var iam_group_sudo=ssh-sudo-group -var iam_group=ssh-group ssh-iam.json`.
1. Open `vars.tf`, set the environment variables specified at the top of the file, and fill in any other variables that
   don't have a default, including the ID of the AMI you built in the previous step.
1. Run `terraform get`.
1. Run `terraform plan`.
1. If the plan looks good, run `terraform apply`.
1. When the templates are done applying, you'll see the IP address of the EC2 Instance. You should now be able to SSH
   to it using your IAM username and the public key you uploaded in step 1: `ssh <IAM-USERNAME>@<SERVER-IP>`.