**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/iam-user-password-policy/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# IAM User Password Policy Example

This is an example of how to use the [iam-user-password-policy module](/modules/iam-user-password-policy) to setup a
policy that governs requirements for all IAM User passwords.

This example will apply the policy in the AWS account and overwrite any other existing policy. 

See the [iam-user-password-policy module](/modules/iam-user-password-policy) for additional details.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `vars.tf` and fill in any variables that don't have a default.
1. Review the hardcoded values in `main.tf`.
1. Run `terraform apply` to apply the password policy.