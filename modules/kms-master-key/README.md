**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/kms-master-key/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# KMS Master Key Module

This Terraform Module creates a new [Customer Master
Key (CMK)](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in [Amazon's Key Management
Service (KMS)](https://aws.amazon.com/kms/) as well as a [Key
Policy](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key_permissions) that controls who has
access to the CMK. 

A CMK is a key managed by AWS that you never see (and can therefore never compromise). You use use a CMK via the AWS API
to encrypt and decrypt small amounts of data and to generate [Data Keys](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) 
that can be used to encrypt and decrypt larger amounts of data.

Using the AWS API with KMS can be clumsy. For a more streamlined experience, try [gruntkms](https://github.com/gruntwork-io/gruntkms).

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See the [kms-master-key example](/examples/kms-master-key) for an example.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.

**Note**: This module creates a Master Key in KMS. Each Master Key costs $1/month, even if you delete it immediately
after. So please be aware that using this module will cost you money!

### CMK Administrators vs. CMK Users
 
This CMK Key Policy declares two levels of access to the CMK:
 
 1. **Key Administrators:** Administrators can *manage* the CMK, including updating the Key Policy, revoking the CMK, and 
    getting additional info about the CMK. Administrators get no permissions to actually use the CMK, for example, with
    encrypt or decrypt operations.
    
 1. **Key Users:** Users can *use* the CMK for encrypt and decrypt operations but cannot manage it.
 
You must have at least one IAM ARN for each user type. Note that this ARN can be an IAM User, IAM Group, or IAM Role. 

## Background

### What is KMS?

[Amazon's Key Management Service (KMS)](https://aws.amazon.com/kms/) is a managed service that makes it easy for you to
create and control the encryption keys used to encrypt your data, and uses Hardware Security Modules (HSMs) to protect
the security of your keys.

### What is a Customer Master Key?

A Customer Master Key (CMK) is a secret key that KMS stores and manages for you. You can use the CMK to encrypt small
amounts of data using the AWS APIs or a tool like [gruntkms](https://github.com/gruntwork-io/gruntkms). It is a "master"
key in the sense that you can also use a CMK to generate a "Data Key" that you can use in your own encryption algorithms 
to encrypt and decrypt large amounts of data. 

When you use a Data Key, you typically store it, encrypted via the CMK, in version control or co-located with your data 
itself, and decrypt it via the AWS API or gruntkms whenever you need to use it to decrypt or encrypt data.

Amazon never grants you access to the CMK itself, only to operations that use the key.

### Managing a Key's Permissions with the Key Policy vs. IAM Policies

When you want to grant a permission on most AWS resources, you attach an [IAM Policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
to an IAM User, IAM Group, or IAM Role. This works well for most resources, but when it comes to CMK's, it means that any
admin-level IAM User has full access to all CMK's. 

But maybe this isn't what you want. For example, suppose your DevOps team has admin-level access to your AWS account, but 
they still shouldn't have access to a `prod` CMK used to encrypt production data. Fortunately, AWS gives us a solution 
for such situations: the CMK Key Policy. 

By default, only the permissions granted in a CMK Key Policy are honored. For example, the CMK Key Policy might 
grant IAM User `jane.doe` the `kms:encrypt` and `kms:decrypt` permissions. But if `john.doe` has an IAM Policy that grants
him those same permissions on the CMK, that IAM Policy will actually have no effect.
 
If you do want to honor IAM Policies for a particular CMK, you can include a setting in the CMK Key Policy that 
grants this permission to IAM. In this case, `jane.doe` will retain her rights granted from the CMK Key Policy, but now
 `john.doe` will have access, too.
 
In general, we recommend using only the CMK Key Policy if possible. This has the benefit of explicitly declaring who has
access to the CMK, versus allowing any possible number of IAM Policy configurations to determine access. But the biggest
downside is that it's now possible to lock yourself out of the CMK, so if you're not confident about your ability to
manage the CMK, you may wish to use IAM policies. In addition, IAM is a central place for managing all permissions, whereas
using just the CMK Key Policy means you now need to update the Key Policy any time the perissions change, which may be 
more onerous.

## TODO

- Explicitly test that granting another AWS resource such as an S3 Bucket privileges on the KMS Key works as expected
  for key users.