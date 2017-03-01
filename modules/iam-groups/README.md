**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/iam-groups/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# A Best-Practices Set of IAM Groups

This Gruntwork Terraform Module sets up a set of IAM Groups that will make sense for most organizations and attaches to 
them a set of IAM Policies (permissions) that make it easier to manage different permissions levels in your AWS account.

If you're not familiar with IAM concepts, start with the [Background Information](#background-information) section as a 
way to familiarize yourself with the terminology.


## Motivation 

In Summer 2014, a company called CodeSpaces that offered "rock solid, secure, and affordable git hosting and project 
management" was forced to shut down after a single rogue employee entered its AWS account and wiped out everything 
([See ArsTechnica Article](http://arstechnica.com/security/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/)). 
The goal of this module is to carefully manage access to your AWS account to reduce the chances of rogue employees or 
external attackers being able to do too much damage.




## How do you use this module?


### Requirements

- You will need to be authenticated to AWS with an account that has `iam:*` permissions. 


### Instructions

Check out the [iam-groups example](../../examples/iam-groups) for a working example.




## Resources Created


### IAM Groups

This module optionally creates the following IAM Groups:

- **full-access:** IAM Users in this group have full access to all resources in the AWS account.
- **billing:** IAM Users in this group can read and write billing settings, but nothing else.
- **developers:** IAM Users in this group have whatever permissions are declared in 
  `var.iam_group_developers_permitted_services`. In addition, these IAM Users have rights to a personal S3 bucket 
  named `<var.iam_group_developers_permitted_services><iam-user-name>`. 
- **read-only:** IAM Users in this group can read all resources in the AWS account but have no write privileges.
- **use-existing-iam-roles:** IAM Users in this group can pass *existing* IAM Roles to AWS resources to which they have 
  been granted access. These IAM Users cannot create *new* IAM Roles, only use existing ones. See 
  [the three levels of IAM permissions](/modules/iam-policies#the-three-levels-of-iam-permissions) for more information.
- **ssh-iam-sudo-users:** IAM Users in this group have SSH access with `sudo` privileges to any EC2 Instance configured
  to use this group to manage SSH logins.
- **ssh-iam-users:** IAM Users in this group have SSH access without `sudo` privileges to any EC2 Instance configured 
  to use this group to manage SSH logins.
- **cross-account-access:** IAM users in these groups can assume an IAM role in another AWS account. This makes
  [cross-account access](https://aws.amazon.com/blogs/security/enable-a-new-feature-in-the-aws-management-console-cross-account-access/),
  easy, where you can have all your users defined in one AWS account (e.g. users) and to grant those users access to 
  certain IAM roles in other AWS accounts (e.g. stage, prod). The IAM groups that are created and which IAM roles they
  have access to is controlled by the variable `var.iam_groups_for_cross_account_access`. 

These represent a standard set of IAM Groups, but your organization may need additional groups. You're welcome to add 
additional IAM Groups outside this module to suit your organization's needs. Is the IAM Group you need a Group most 
teams would need? Let us know at support@gruntwork.io and maybe we'll consider adding it to the module itself.


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

### Enable Your Billing IAM Group

By default, only the root AWS account has access to billing information. To enable the billing IAM Group above or 
otherwise enable IAM Users to access the billing console:

1. Select "My Account" from the top right of the AWS Web Console.

   ![Screenshot](_docs/my-account.png)

1. You'll be taken to the "Dashboard" page. Now scroll down until you see the below heading and check the box:

   ![Screnshot](_docs/iam-user-access-to-billing.png)  





## Background Information

For background information on IAM, IAM users, IAM policies, and more, check out the [background information docs in
the iam-policies module](/modules/iam-policies#background-information).




## TODO

Are we missing any functionality? Let us know by emailing info@gruntwork.io!