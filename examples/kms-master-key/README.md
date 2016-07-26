**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/kms-master-key/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# KMS Master Key Example

This is an example of how to create a [Customer Master
Key (CMK)](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in [Amazon's Key Management
Service (KMS)](https://aws.amazon.com/kms/) using the [kms-master-key module](/modules/kms-master-key). You can use a
CMK to encrypt and decrypt small amounts of data and to generate [Data
Keys](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) that can be used to encrypt and
decrypt larger amounts of data.

You can use KMS via the AWS API, CLI, or, for a more streamlined experience, you can use
[gruntkms](https://github.com/gruntwork-io/gruntkms).

## Quick start

**Note**: These templates create a Master Key in KMS. Each Master Key costs $1/month, even if you delete it immediately
after. So please be aware that applying this example will cost you money!

To try these templates out you must have Terraform installed (minimum version: `0.6.11`):

1. Open `vars.tf`, set the environment variables specified at the top of the file, and fill in any other variables that
   don't have a default.
1. Run `terraform get`.
1. Run `terraform plan`.
1. If the plan looks good, run `terraform apply`.