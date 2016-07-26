**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/kms-master-key/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# KMS Master Key Module

This Terraform Module creates a new [Customer Master
Key (CMK)](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in [Amazon's Key Management
Service (KMS)](https://aws.amazon.com/kms/) as well as a [Key
Policy](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key_permissions) that controls who has
access to the CMK. You can use a CMK to encrypt and decrypt small amounts of data and to generate [Data
Keys](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) that can be used to encrypt and
decrypt larger amounts of data.

You can use KMS via the AWS API, CLI, or, for a more streamlined experience, you can use
[gruntkms](https://github.com/gruntwork-io/gruntkms).

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See the [kms-master-key example](/examples/kms-master-key) for an example.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.

**Note**: This module creates a Master Key in KMS. Each Master Key costs $1/month, even if you delete it immediately
after. So please be aware that using this module will cost you money!

## What is KMS?

[Amazon's Key Management Service (KMS)](https://aws.amazon.com/kms/) is a managed service that makes it easy for you to
create and control the encryption keys used to encrypt your data, and uses Hardware Security Modules (HSMs) to protect
the security of your keys.

## What is a Customer Master Key?

A Customer Master Key (CMK) is a secret key that KMS stores and manages for you. You can use the CMK to encrypt small
amounts of data using the AWS APIs or a tool like [gruntkms](https://github.com/gruntwork-io/gruntkms). You can also use
a CMK to generate a Data Key that you can use in your own encryption algorithms to encrypt and decrypt large amounts of
data. You typically store the Data Key, encrypted via the CMK, in version control, and decrypt it via the AWS API or
gruntkms whenever you need to use it to decrypt or encrypt data.