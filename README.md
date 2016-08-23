**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Security Modules

This repo contains modules for setting up best practices for managing secrets, credentials, and servers:

* [ssh-iam](/modules/ssh-iam): This module contains an app called `ssh-iam` that allows you to manage SSH access to
  your EC2 Instances using AWS IAM. Developers can upload public SSH Keys to their IAM user accounts and `ssh-iam`
  will allow them to SSH to EC2 Instances using their IAM user name and SSH key for authentication.
* [auto-update](/modules/auto-update): This module can configure a Linux server to automatically install security
  updates.
* [kms-master-key](/modules/kms-master-key): This Terraform Module creates a new [Customer Master Key
  (CMK)](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in [Amazon's Key Management
  Service (KMS)](https://aws.amazon.com/kms/) as well as a [Key
  Policy](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key_permissions) that controls who has
  access to the CMK. You can use a CMK to encrypt and decrypt small amounts of data and to generate [Data
  Keys](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) that can be used to encrypt and
  decrypt larger amounts of data.

Click on each module above to see its documentation. Head over to the [examples](/examples) folder for examples.

## What is a module?

At [Gruntwork](http://www.gruntwork.io), we've taken the thousands of hours we spent building infrastructure on AWS and
condensed all that experience and code into pre-built **packages** or **modules**. Each module is a battle-tested,
best-practices definition of a piece of infrastructure, such as a VPC, ECS cluster, or an Auto Scaling Group. Modules
are versioned using [Semantic Versioning](http://semver.org/) to allow Gruntwork clients to keep up to date with the
latest infrastructure best practices in a systematic way.

## How do you use a module?

Most of our modules contain either:

1. [Terraform](https://www.terraform.io/) code
1. Scripts & binaries

#### Using a Terraform Module

To use a module in your Terraform templates, create a `module` resource and set its `source` field to the Git URL of
this repo. You should also set the `ref` parameter so you're fixed to a specific version of this repo, as the `master`
branch may have backwards incompatible changes (see [module
sources](https://www.terraform.io/docs/modules/sources.html)).

For example, to use `v1.0.8` of the ecs-cluster module, you would add the following:

```hcl
module "ecs_cluster" {
  source = "git::git@github.com:gruntwork-io/module-ecs.git//modules/ecs-cluster?ref=v1.0.8"

  // set the parameters for the ECS cluster module
}
```

*Note: the double slash (`//`) is intentional and required. It's part of Terraform's Git syntax (see [module
sources](https://www.terraform.io/docs/modules/sources.html)).*

See the module's documentation and `vars.tf` file for all the parameters you can set. Run `terraform get -update` to
pull the latest version of this module from this repo before runnin gthe standard  `terraform plan` and
`terraform apply` commands.

#### Using scripts & binaries

You can install the scripts and binaries in the `modules` folder of any repo using the [Gruntwork
Installer](https://github.com/gruntwork-io/gruntwork-installer). For example, if the scripts you want to install are
in the `modules/ecs-scripts` folder of the https://github.com/gruntwork-io/module-ecs repo, you could install them
as follows:

```bash
gruntwork-install --module-name "ecs-scripts" --repo "https://github.com/gruntwork-io/module-ecs" --tag "0.0.1"
```

See the docs for each script & binary for detailed instructions on how to use them.

## Server security

For an intro to server security, check out the following guides:

* [My First 10 Minutes On a Server - Primer for Securing
  Ubuntu](http://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/)
* [2016 Guide to User Data Security](https://www.inversoft.com/guides/2016-guide-to-user-data-security)
* [7 Security Measures to Protect Your
  Servers](https://www.digitalocean.com/community/tutorials/7-security-measures-to-protect-your-servers)
* The [Open Web Application Security Project](https://www.owasp.org/index.php/Main_Page), especially their handy
  cheat sheets on [Password Storage](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet), [Session
  Management](https://www.owasp.org/index.php/Session_Management_Cheat_Sheet), [SQL
  Injection](https://www.owasp.org/index.php/SQL_Injection), and
  [XSS](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet).
* [What should every programmer know about
  security?](http://stackoverflow.com/questions/2794016/what-should-every-programmer-know-about-security)

## Developing a module

#### Versioning

We are following the principles of [Semantic Versioning](http://semver.org/). During initial development, the major
version is to 0 (e.g., `0.x.y`), which indicates the code does not yet have a stable API. Once we hit `1.0.0`, we will
follow these rules:

1. Increment the patch version for backwards-compatible bug fixes (e.g., `v1.0.8 -> v1.0.9`).
2. Increment the minor version for new features that are backwards-compatible (e.g., `v1.0.8 -> 1.1.0`).
3. Increment the major version for any backwards-incompatible changes (e.g. `1.0.8 -> 2.0.0`).

The version is defined using Git tags.  Use GitHub to create a release, which will have the effect of adding a git tag.

#### Tests

See the [test](/test) folder for details.

## License

Please see [LICENSE.txt](/LICENSE.txt) for details on how the code in this repo is licensed.