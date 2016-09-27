**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/iam-groups/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# A Best-Practices Set of IAM Group Permissions

This Gruntwork Terraform Module sets up a set of IAM Groups that will make sense for most organizations and attaches to 
them a set of IAM Policies (permissions) that make it easier to manage different permissions levels in your AWS account.

If you're not familiar with IAM concepts, start with the [Background Information](#background-information) section as a 
way to familiarize yourself with the terminology.

### Motivation 

In Summer 2014, a company called CodeSpaces that offered "rock solid, secure, and affordable git hosting and project 
management" was forced to shut down after a single rogue employee entered its AWS account and wiped out everything 
([See ArsTechnica Article](http://arstechnica.com/security/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/)). The goal of this module is to carefully manage access to your AWS account to reduce the chances of rogue employees or external attackers being able to do too much damage.

## How do you use this module?

### Requirements

- You will need to be authenticated to AWS with an account that has most `iam:*` permissions. 

### Instructions

Check out the [iam-groups example](../../examples/iam-groups) for a working example.

## Resources Created

### IAM Groups

This module creates the following IAM Groups:

- **admin:** IAM Users in this group have full access to all resources in the AWS account.
- **billing:** IAM Users in this group can read and write billing settings, but nothing else.
- **read-only:** IAM Users in this group can read all resources in the AWS account but have no write privileges.
- **use-existing-iam-roles:** IAM Users in this group can pass *existing* IAM Roles to AWS resources to which they have 
  been granted access. These IAM Users cannot create *new* IAM Roles, only use existing ones. See [below](#the-three-levels-of-iam-permissions)
  for more information.

These represent a standard set of IAM Groups, but your organization may need additional groups (e.g. "developers"). As
shown in the [iam-groups example](../../examples/iam-groups), you are encouraged to create any additional IAM Groups 
that make sense. Use the above IAM Groups as a good starting point.

### IAM Policies

This module creates the following IAM Policies:

- **iam-user-self-mgmt:** This IAM Policy grants permission to an IAM User to manage her own IAM User account. This
  includes resetting the IAM User password, and generating AWS account credentials. It also grants permission to list 
  other IAM Users, but not to view any information about them.

This IAM Policy should be attached to any IAM Group that does not already grant all IAM permissions. For example, if an
IAM User does not have the ability to pass an IAM Role to an EC2 Instance, that IAM User will be unable to manage his 
own account unless this IAM Policy is attached to his account. 

## Resources NOT Created

### IAM Users

This module does not create any IAM Users, nor assign any existing IAM Users to IAM Groups.

You should continue to use the AWS Web Console to manually create and manage users. Although it's possible to manage IAM 
Users with Terraform, there are some downsides, including credentials leaked in terraform state files, and a chicken-and-egg
problem about whether IAM Users should use Terraform to create more IAM Users.

### IAM Roles

This module does not create any IAM Roles. Those should be created with Terraform, but in separate templates in the 
context of your specific needs, not as a generic set of roles.

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

### Enable Your Billing IAM Group

By default, only the root AWS account has access to billing information. To enable the billing IAM Group above or 
otherwise enable IAM Users to access the billing console:

1. Select "My Account" from the top right of the AWS Web Console.

   ![Screenshot](_docs/my-account.png)

1. You'll be taken to the "Dashboard" page. Now scroll down until you see the below heading and check the box:

   ![Screnshot](_docs/iam-user-access-to-billing.png)  

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

### Gotcha's 

#### Avoid the Terraform resource `aws_iam_policy_attachment`

Be very careful about using the [aws.aws_iam_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_policy_attachment.html)
Terraform resource. Using this resource with a Managed IAM Policy will remove any other IAM Users, IAM Groups, or IAM 
Roles that have attached that Managed Policy.

Instead, use these Terraform resources so you don't have to worry about this problem:

-  [aws_iam_group_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_group_policy_attachment.html)
-  [aws_iam_role_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_role_policy_attachment.html)
-  [aws_iam_user_policy_attachment](https://www.terraform.io/docs/providers/aws/r/iam_user_policy_attachment.html)

# TODO

- Think through how IAM Users can be forced to use MFA.
- Add tests to validate that the `iam-user-self-mgmt` IAM Policy allows updates to a user's MFA device as expected.