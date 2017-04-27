**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/iam-user-password-policy/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Set a Password Policy for IAM Users

This Gruntwork Terraform Module sets the [AWS Account Password Policy](
http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html) that will govern password
requirements for IAM Users.



## Motivation

This module adds no value beyond directly using the [aws_iam_account_password_policy](
https://www.terraform.io/docs/providers/aws/r/iam_account_password_policy.html), except that having a standardized 
module supported by Gruntwork enables you to easily invoke this Terraform resource using Terragrunt's functionality of
downloading a module and setting values with nothing more than a `terraform.tfvars` file.



## How do you use this module?

### Requirements

- You will need to be authenticated to AWS with an account that has `iam:*` permissions. 

### Instructions

Check out the [iam-user-password-policy example](../../examples/iam-user-password-policy) for a working example.



## Resources Created

### IAM User Password Policy

This module will apply the desired password policy to the given AWS account. Note that this will overwrite any existing
password policy you already have in place!



## TODO

Are we missing any functionality? Let us know by emailing info@gruntwork.io!