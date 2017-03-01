**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/cross-account-iam-roles/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Cross-account IAM roles example

This is an example of how to use the [cross-account-iam-roles module](/modules/cross-account-iam-roles) to create IAM
roles that users in other AWS accounts can assume to get access to this AWS account.

Note that while these templates allow other accounts to access this account, you will also need to add IAM policies in 
those other accounts for users to actually be able to switch between accounts. See the 
[iam-groups module](/modules/iam-groups) for how to create those policies.
 



## Quick start

To try these templates out you must have Terraform installed:

1. Open `vars.tf`, specify the environment variables mentioned at the top of the file, and fill in any variables that 
   don't have a default.
1. Run `terraform apply` to create the IAM Roles. 
1. This module will output the ARNs of these IAM roles as well as convenient sign-in URLs to assume those roles.
1. In a separate AWS account, use the [iam-groups module](/modules/iam-groups) to give users permissions to assume the
   IAM Role ARNs from the previous step.
1. Use the convenient sign-in URLs to switch roles. See the [How to switch between accounts
   documentation for more details](/modules/cross-account-iam-roles#how-to-switch-between-accounts).   