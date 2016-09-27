**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/iam-groups/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# IAM Groups Setup Example

This is an example of how to use the [iam-groups module](/modules/iam-groups) to create an effective set of IAM Groups.
See the iam-groups module documentation for additional detail.

This example will create a common set of IAM Groups as defined in the `iam-groups` module, an additional "custom" IAM 
Group defined in the `main.tf` file, and a [Customer Managed IAM Policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html) 
(also in the `iam-groups` module) that can be used to grant limited-access users the permissions to manage their own IAM 
account. 

See the [iam-groups module](/modules/iam-groups) for additional details.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `vars.tf` and fill in any variables that don't have a default.
1. Review the "custom IAM Groups" in `main.tf` and decide if you wish to keep or remove them. To remove them, delete 
   the corresponding Terraform resources. 
1. Run `terraform apply` to create the IAM Groups.
1. Now log into the AWS Web Console and assign some IAM Users to IAM Groups.