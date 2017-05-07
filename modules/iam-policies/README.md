**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/iam-policies/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# A Best-Practices Set of IAM Policy Documents

This Gruntwork Terraform Module sets up a set of IAM Policy Documents that can be used in IAM users, groups, and roles.
The documents represent a reasonable collection of permissions that will make sense for most organizations for 
managing different permissions levels in your AWS account.

Note that these documents are Terraform [data sources](https://www.terraform.io/docs/configuration/data-sources.html),
so they don't create anything themselves and are not intended to be used on their own. The way to use them is to take 
the outputs from this module (which are all JSON IAM documents) and plug them into other Terraform resources, such 
as `aws_iam_policy`, `aws_iam_user_policy`, `aws_iam_group_policy`, and `aws_iam_role_policy`. See the 
[iam-groups](../iam-groups) and [corss-account-iam-roles](../cross-account-iam-roles) modules for examples.

If you're not familiar with IAM concepts, start with the [Background Information](#background-information) section as a 
way to familiarize yourself with the terminology.




## Motivation 

In Summer 2014, a company called CodeSpaces that offered "rock solid, secure, and affordable git hosting and project 
management" was forced to shut down after a single rogue employee entered its AWS account and wiped out everything 
([See ArsTechnica Article](http://arstechnica.com/security/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/)). 
The goal of this module is to carefully manage access to your AWS account to reduce the chances of rogue employees or 
external attackers being able to do too much damage.




## How do you use this module?

You use this module like any other [Terraform module](https://www.terraform.io/docs/modules/usage.html):

```hcl
module "iam_policies" {
  source = "../iam-policies"

  aws_account_id = "123456789012"
  should_require_mfa = true
  iam_group_developers_s3_bucket_prefix = "my-company.dev"
  iam_group_developers_permitted_services = ["ec2:*"]
}
```

You can now access all of the documents as outputs on the module. E.g.:

```hcl
resource "aws_iam_group_policy" "full_access" {
  name = "full-access"
  group = "${aws_iam_group.full_access.id}"
  policy = "${module.iam_policies.full_access}"
}
```

You can find the full list of outputs in [outputs.tf](outputs.tf).

Check out the [iam-groups](../iam-groups) and [corss-account-iam-roles](../cross-account-iam-roles) modules for example code.




## IAM Policy Documents created

This module creates the following IAM Policy documents:

- **full-access:** provides full access to all resources in the AWS account.

- **billing:** provides read and write billing settings, but nothing else.

- **developers:** provides whatever permissions are declared in `var.dev_permitted_services`. 
  In addition, creates permissions for a personal S3 bucket named `<var.dev_permitted_services><iam-user-name>`. 

- **read-only:** provides read access to all resources in the AWS account but have no write privileges.

- **use-existing-iam-roles:** allows passing *existing* IAM Roles to AWS resources to which you have been granted 
  access. Does not allow creating *new* IAM Roles. See [below](#the-three-levels-of-iam-permissions) for more 
  information.

- **iam-user-self-mgmt:** provides permission to manage your own IAM User account. This includes resetting the IAM User 
  password, and generating AWS account credentials. It also grants permission to list other IAM Users, but not to view 
  any information about them.

- **allow_access_to_other_accounts:** provides permission to assume an IAM role in another AWS account. This makes
  [cross-account access](https://aws.amazon.com/blogs/security/enable-a-new-feature-in-the-aws-management-console-cross-account-access/),
  easy, where you can have all your users defined in one AWS account (e.g. users) and to grant those users access to 
  certain IAM roles in other AWS accounts (e.g. stage, prod). The documents that are created and which IAM roles they
  have access to is controlled by the variable `var.allow_access_to_other_account_arns`.

- **allow_access_to_all_other_accounts:** provides permission to assume an IAM role in all the external AWS accounts 
  specified in `var.allow_access_to_other_account_arns`.

- **allow_access_from_other_accounts:** allows users from other AWS accounts to assume specific roles in this account. 
  This makes [cross-account access](https://aws.amazon.com/blogs/security/enable-a-new-feature-in-the-aws-management-console-cross-account-access/),
  easy, where you can have all your users defined in one AWS account (e.g. users) and to grant those users access to 
  certain IAM roles in other AWS accounts (e.g. stage, prod). The documents that are created and which IAM roles they
  have access to is controlled by the variable `var.allow_access_from_other_account_arns`.

- **ssh_iam_permissions**: provides the permissions [ssh-iam](/modules/ssh-iam) needs to validate SSH keys. 

- **auto_deploy_permissions**: provides the permissions in `var.auto_deploy_permissions` to do automated deployment.
  The primary use case is to add these permissions to the IAM role of a CI server (e.g. Jenkins).

- **allow_auto_deploy_from_other_accounts**: allows the IAM ARNs in `var.allow_auto_deploy_from_other_account_arns` to
  do automated deployment using the permissions in `var.auto_deploy_permissions`. The primary use cases is to allow a
  CI server (e.g. Jenkins) in another AWS account to do automated deployments in this AWS account.

## Additional Guidelines


### The Root AWS Account

When you first create your AWS account, only the [root AWS account](http://docs.aws.amazon.com/general/latest/gr/root-vs-iam.html)
exists. While logged in as the root user, your first and only step should be to create a small number of IAM Users and 
assign them the `FullAccess` [AWS managed policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html)
so that they can then configure your AWS account as needed using their IAM User accounts.

Now delete the root account's access keys (the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) and store the root AWS 
account password in a very safe place. You might also consider setting up a Multi-Factor Authentication 
(MFA) device on your root account such as [Authy](https://www.authy.com/) or [Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator).

Note that AWS only allows a single MFA device per account. This means that if you choose to use MFA on your root account,
you should save the initializing QR code along with the password! Otherwise, you may need to contact AWS to restore access
to your account. Another alternative is to initialize more than one device against the same MFA code.

Or for some interesting possibilities, see the Authy article around ["multi multi-factor" authentication](https://www.authy.com/blog/multi-multi-factor-authentication).




## Background Information


### What is IAM?

AWS Identity and Access Management (IAM) is a set of features in every AWS account that can be used to manage and secure
human users, human groups, and AWS Resources.


#### IAM Users 

Human users access the AWS Web Console or AWS API as **IAM Users.** Usually, an IAM User corresponds to a single human,
but sometimes you create IAM Users for machines, like a `circleci` IAM User. 

It's convenient to be able to differentiate between different types of IAM Users, so we recommend using some kind of 
common prefix for different IAM User types. For example:

- `jane.doe` (an employee of your org)
- `_machine.jenkins` (perhaps machine users are denoted with a `_`)
- `_contractor.john.doe` (contractors are called out explicitly)

IAM Users can assert their permissions via the AWS Web Console or AWS API. The [awscli](https://aws.amazon.com/cli/) and
various AWS SDKs all just implement performant ways of connecting to the AWS API.


#### IAM Policies

Once you've created an IAM User, that user has no permissions to do anything. You must grant that user specific permissions 
by attaching one or more **IAM Policies**. 

An IAM Policy is simply a JSON document that declares a set of permissions. For example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:*",
        "lambda:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```

This particular IAM Policy "Allows" all permissions on all `rds`, `lambda` and `ec2` resources.

Each AWS resource type has a set of IAM permissions associated with it. For example, for EC2 Instances, some of the IAM
permissions include:

- `ec2:CreateTags`
- `ec2:DeleteTags`
- `ec2:StartInstances`
- `ec2:StopInstances`  

So in the above example IAM Policy, the action `ec2:*` actually says that this Policy grants *all* permissions on EC2
Resources. It could have granted this permission only to certain EC2 Instances, but thanks to `Resource: "*"`, this 
permission applies to all EC2 resources.

For more on IAM Policies, see the [official docs](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html).


#### IAM Groups

You might be tempted to directly attach permissions to IAM Users, but it's a best practice to instead grant permissions 
through **IAM Groups** by adding IAM Users to one or more IAM Groups. 

Using IAM Groups means keeping track of permissions on a small number of groups, rather than a potentially large number of 
IAM Users.


#### IAM Roles

Sometimes AWS resources themselves need permissions on other AWS resources. For example, an EC2 Instance or Lambda Function
might need to access a file in an S3 Bucket. While you could hard-code your IAM User credentials in these resources, 
hard-coding credentials anywhere is almost always a bad idea since they are now discoverable, non-auditable, and more
likely to leak.

Fortunately, AWS provides an alternative, **IAM Roles.** IAM Roles can be granted permissions via IAM Policies, just 
like with IAM Users and IAM Groups. In addition, an AWS Resource can "assume" an IAM Role and inherit all its permissions.
The process of assuming an IAM Role is handled transparently and securely by AWS.  


### The Three Levels of IAM Permissions

Why do we have a `use-existing-iam-roles` IAM Group? It's because it represents an "intermediate" level of permissions 
on the AWS account. So what's an "intermediate" level of permissions? 

A key insight in configuring IAM Policies is that **an IAM User who has permission to create other permissions can give 
herself any arbitrary permissions and thereby become the equivalent of an admin user.** For this reason, we must limit 
the permission to create permissions.

The following hierarchy is a conceptual model that proposes one such way. Consider this a way of thinking about IAM 
Permissions, not an exact prescription for implementation:

1. **Full IAM Permissions** -- This user is equivalent to root and can do anything in the AWS account, including creating
   new IAM Roles.

   *Corresponds to the "admin" IAM Group above.*

1. **Leverage Existing IAM Roles** -- This user cannot create a *new* IAM Role, but can leverage *existing* IAM Roles
   when launching AWS resources like an EC2 Instance, Lambda job, or ECS Task.

   For example, when launching an EC2 Instance, this user can't create her own IAM Role for that EC2 Instance, but she 
   can choose among existing IAM Roles. Presumably, no IAM Role would itself have the ability to create permissions. 

   *Corresponds to any non-admin IAM Group plus the "use-existing-iam-roles" group above. For example, an IAM User would
   belong both to an IAM Group that grants certain permissions (e.g. launch EC2 Instances) and also to the "use-existing-iam-role" 
   group.*

1. **Leverage No IAM Roles at All** -- This user cannot pass any IAM Roles to any AWS resources, but may have other 
   permissions with respect to AWS resources granted by other IAM Policies or IAM Groups.

   *Corresponds to an IAM User belonging to a custom IAM Group an organization might create *without* also belonging
   to the "use-existing-iam-roles" group above.*

What this model tells us is that all IAM Users must fall into one of these three categories. As an example, if an IAM
User requires the ability to define IAM Roles as part of her job, there's no point limiting this IAM User's permissions.
Instead, just consider them an admin user.


### How do you know what to include in an IAM Policy?

We use a combination of our own personal experience and the [AWS Managed Policies for Job Functions](https://aws.amazon.com/blogs/security/how-to-assign-permissions-using-new-aws-managed-policies-for-job-functions/)
as a reference. Note that none of the AWS Managed Policies allows us to directly specify a `condition` property, which 
means we can't enforce multi-factor authentication (MFA) on these policies. As a result, we've created our own policies,
written automated tests for them, and of course will continue to maintain them over time.


### Gotcha's 

#### Avoid the Terraform resource `aws_iam_policy_attachment`

Be very careful about using the [aws.aws_iam_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_policy_attachment.html)
Terraform resource. Using this resource with a Managed IAM Policy will remove any other IAM Users, IAM Groups, or IAM 
Roles that have attached that Managed Policy.

Instead, use these Terraform resources so you don't have to worry about this problem:

-  [aws_iam_group_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_group_policy_attachment.html)
-  [aws_iam_role_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_role_policy_attachment.html)
-  [aws_iam_user_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_user_policy_attachment.html)




## TODO

Are we missing any functionality? Let us know by emailing info@gruntwork.io!