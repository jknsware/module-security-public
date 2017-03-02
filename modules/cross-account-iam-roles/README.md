**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/cross-account-iam-roles/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# A best-practices set of IAM roles for cross-account access

This module can be used to allow IAM users from other AWS accounts to access your AWS accounts (i.e. [cross-account
access](https://aws.amazon.com/blogs/security/enable-a-new-feature-in-the-aws-management-console-cross-account-access/)). 
This allows you to define each environment (mgmt, stage, prod, etc) in a separate AWS account, your IAM users in a 
single account, and to allow those users to easily switch between accounts with a single set of credentials.


If you're not familiar with IAM concepts, start with the [Background Information](#background-information) section as a 
way to familiarize yourself with the terminology.




## How do you use this module?

To set up cross-account access, you need to:

1. [Create IAM roles in one account](#create-iam-roles-in-one-account)
1. [Create permissions to assume the IAM roles in other accounts](#create-permissions-to-assume-the-iam-roles-in-other-accounts)


### Create IAM roles in one account

If you want to allow users in AWS account A to access AWS account B, use this module in AWS account B to create IAM 
roles that specify which services those users may access. Check out the [cross-account-iam-roles 
example](/examples/cross-account-iam-roles) for a working sample code of how to use this module.


### Create permissions to assume the IAM roles in other accounts

Now that you have created IAM roles in account B, you need to give users in account A permission to use that role. The
[iam-groups module](/modules/iam-groups) can automatically create IAM groups that grant these permissions. 

In account A, do the following:

1. Take the ARNs of the IAM roles you created in account B and plug them into the 
   `var.iam_groups_for_cross_account_access` input variable of the `iam-groups` module in account A:
   
    ```hcl
    module "iam_groups" {
      source = "git::git@github.com:gruntwork-io/module-security.git//modules/iam-groups?ref=v1.0.8"
   
      iam_groups_for_cross_account_access = "${map("account-b-read-only-access", "arn:aws:iam::1234567901234:role/allow-read-only-access-from-other-accounts")}"
   
      # ... (other params omitted) ...
    }
    ```
      
1. Run `terraform apply` on the `iam-groups` module.

1. Add users from account A to the corresponding groups that get created (e.g. `account-b-read-only-access`).

1. Users in account A can now "switch" to the roles in account B as described in [How to switch between 
   accounts](#how-to-switch-between-accounts).




## Resources Created

This module creates the following IAM roles (all optional):

* **allow-read-only-access-from-other-accounts**: Users from the accounts in 
  `var.allow_read_only_access_from_other_account_arns` will get read-only access to all services in this account.

* **allow-billing-access-from-other-accounts**: Users from the accounts in 
  `var.allow_billing_access_from_other_account_arns` will get full (read and write) access to the billing details for 
  this account.

* **allow-ssh-iam-access-from-other-accounts**: Users (or more likely, EC2 Instances) from the accounts in 
  `var.allow_ssh_iam_access_from_other_account_arns` will get ead access to IAM Groups and public SSH keys. This is
  useful to allow [ssh-iam](/modules/ssh-iam) running on EC2 Instances in other AWS accounts to validate SSH 
  connections against IAM users defined in this AWS account.

* **allow-dev-access-from-other-accounts**: Users from the accounts in `var.allow_dev_access_from_other_account_arns` 
  will get full (read and write) access to the services in this account specified in `var.dev_permitted_services`.

* **allow-full-access-from-other-accounts**: Users from the accounts in `var.allow_full_access_from_other_account_arns` 
  will get full (read and write) access to all services in this account.
  



## How to switch between accounts


### Switching in the AWS console

Check out the [AWS Switching to a Role (AWS Console)
documentation](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-console.html).

Note that this module automatically outputs the convenient sign-in URLs to quickly switch to a given role. The outputs
are named `allow_XXX_access_sign_in_url`, where `XXX` is one of `read-only`, `billing`, `dev`, or `full`.


### Switching with CLI tools (including Terraform)

Check out the [AWS Switching to a Role (AWS Command Line Interface)
documentation](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-cli.html).




## Background Information

For background information on IAM, IAM users, IAM policies, and more, check out the [background information docs in
the iam-policies module](/modules/iam-policies#background-information).
