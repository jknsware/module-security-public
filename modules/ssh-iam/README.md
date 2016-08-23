**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/ssh-iam/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# SSH IAM

This module contains an app called `ssh-iam` that allows you to manage SSH access to your EC2 Instances using AWS
[Identity and Access Management (IAM)](https://aws.amazon.com/iam/). Developers can upload public SSH Keys to their
IAM user accounts and `ssh-iam` will allow them to SSH to EC2 Instances using their IAM user name and SSH key for
authentication.

Features:

* With `ssh-iam`, each developer uses their own SSH keys to connect to servers (instead of a single, shared Key Pair).
* If a developer's SSH key gets compromised, they can use IAM to revoke the old public key and upload a new one and
  `ssh-iam` will pick up the update on the very next login.
* If a developer leaves your company, as soon as you remove their IAM user, they will no longer be able to SSH to your
  servers.
* `ssh-iam` can automatically sync user accounts from IAM to your servers, so each developer can have their own user
  name (e.g. "susan", "jim") rather than everyone using a shared user (e.g. "ubuntu", "ec2-user").
* `ssh-iam` supports all major Linux distros.

## Setup instructions

To use `ssh-iam`, you need to do the following:

1. [Upload public SSH Keys to IAM](#upload-public-ssh-keys-to-iam)
1. [Install ssh-iam on your servers](#install-ssh-iam-on-your-servers)
1. [Set up IAM permissions](#set-up-iam-permissions)
1. [Test it out](#test-it-out)

See the [ssh-iam example](/examples/ssh-iam) for fully working code samples.

#### Upload public SSH Keys to IAM

Login to the [IAM console](https://console.aws.amazon.com/iam/home) and click on your User Name in the Users section.
At the bottom of the Security Credentials tab, click the "Upload SSH public key" button, and enter the contents of
your public SSH key (which is typically stored in `~/.ssh/id_rsa.pub`):

![Upload public SSH Key to IAM](./_docs/iam-upload-ssh-key.png)

#### Install ssh-iam on your servers

`ssh-iam` consists of a single binary. The easiest way to get it onto your servers is to use the [Gruntwork
Installer](https://github.com/gruntwork-io/gruntwork-installer):

```
gruntwork-install --binary-name ssh-iam --repo https://github.com/gruntwork-io/module-security --tag v0.0.3
```

Alternatively, you can download the binary from the [Releases
Page](https://github.com/gruntwork-io/module-security-public/releases).

Once the binary is on the server, run the `install` command:

```
ssh-iam install --iam-group ssh-iam-access
```

The `install` command does two things:

1. Create a cron job to run `ssh-iam sync-users` to periodically (by default, every 30 minutes) sync user accounts
   from the IAM Group specified via the `--iam-group` option.
1. Configure SSH to use `ssh-iam print-keys` to authenticate login attempts using public keys in IAM.

See the [CLI docs](#cli-docs) for the full list of available options.

#### Set up IAM permissions

`ssh-iam` retrieves info from IAM using the AWS API. To use the API, `ssh-iam` needs:

1. AWS credentials: the best way to provide credentials is to attach an [IAM
   Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) to the EC2 Instance.
1. IAM permissions: `ssh-iam` needs an [IAM
   Policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) that allows the `iam:GetGroup`,
   `iam:ListSSHPublicKeys`, and `iam:GetSSHPublicKey` actions.

Check out the [ssh-iam example](/examples/ssh-iam) for sample code.

#### Test it out

You should now be able to SSH to your servers using your IAM user name and public key. For example, if you installed
`ssh-iam` on an EC2 Instance with IP address `11.22.33.44` and your IAM user name is `grunt`, you can now connect to
the server as follows:

```
ssh grunt@11.22.33.44
```

That's all there is to it! No Key Pair files or passwords to worry about and as you update users in IAM, your servers
will sync up automatically!

To learn more, check out the [How it works](#how-it-works) docs to see what's happening under the hood, and the
[CLI docs](#cli-docs) to learn all the parameters and options you can tweak.

## How it works

`ssh-iam` consts of two main pieces:

1. [Syncing users from IAM](#syncing-users-from-iam)
1. [Authenticating requests with public keys from IAM](#authenticating-requests-with-public-keys-from-iam)

#### Syncing users from IAM

When you run `ssh-iam install`, it adds a cron job that periodically runs the `ssh-iam sync-users` command. The
`sync-users` command does the following:

1. Fetch the users from the IAM Group specified via the `--iam-group` option and for each one, create an OS user on
   the EC2 Instance. All of these OS users get added to the OS group `ssh-iam-non-sudo-group`.
1. Fetch the users from the IAM Group specified via the `--iam-group-sudo` option and for each one, create an OS user
   that has sudo privileges on the EC2 Instance. All of these OS users get added to the OS group `ssh-iam-sudo-group`.
1. Remove any users in the `ssh-iam-non-sudo-group` and `ssh-iam-sudo-group` groups that are not in the corresponding
   IAM group.

Note that the requirements for OS user names are stricter than IAM user names. Therefore, the OS user name created by
the `sync-users` command may not exactly match the IAM user name. In particular, OS user names must start with a
letter, followed by letters, numbers, underscores, and dashes, and be no more than 32 characters long. Therefore, we:

1. Drop anything after the @ symbol in email addresses (e.g. "foo@bar.com" --> "foo").
1. Replace whitespace and periods with underscores (e.g. "foo.bar baz" --> "foo_bar_baz").
1. Drop all other illegal characters (e.g. "foo$@%,!bar" --> "foobar").
1. Drop any leading punctuation or numbers (e.g. "_123foo" --> "foo").
1. Truncate all strings over 32 characters long.

We store the original IAM user name in the comment field of the OS user so we can retrieve it when the user tries to
authenticate with their public key.

#### Authenticating requests with public keys from IAM

When you run `ssh-iam install`, it modifies the `AuthorizedKeysCommand` in `/etc/ssh/sshd_config`. When a user tries
to connect via SSH, SSH will execute the `AuthorizedKeysCommand` (passing it the username as the first argument) and
whatever text that command writes to stdout will be used as the `authorized_keys`—that is, the list of public keys that
are allowed to connect to the server.

We take advantage of this `AuthorizedKeysCommand` by configuring it to run `ssh-iam print-keys`, which looks up the
user's public keys in IAM and prints all the active keys to stdout. That way, all a user has to do is upload their
public keys to IAM and they will be able to authenticate to any server with `ssh-iam`.

Note that SSH is very picky about the program it executes for the `AuthorizedKeysCommand` (See the [sshd_config
docs](http://man.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/sshd_config.5) for details):

* The program must not use any custom arguments, must be owned by root, must not be writable by group or others, and
  must be specified by an absolute path. To meet these requirements, we create a script under `/opt/ssh-iam` that runs
  `ssh-iam print-keys`.
* The program must be executed under a user specified via the `AuthorizedKeysCommandUser` configuration. By default,
  we set this configuration to the user that runs `ssh-iam install`.

## CLI docs

The `ssh-iam` CLI supports the following commands:

1. [print-keys](#print-keys-command)
1. [sync-users](#sync-users-command)
1. [install](#install-command)

You can run `ssh-iam --help` at any time to see the CLI docs.

#### print-keys command

Usage: `ssh-iam print-keys USERNAME`

Description: Find the public keys stored for the specified IAM user and print them to stdout.

Arguments:

* `USERNAME` (required): Look up the public keys for this IAM user name.

Examples:

`ssh-iam print-keys grunt`

#### sync-users command

Usage: `ssh-iam sync-users [OPTIONS]`

Description: Sync the user accounts on this system with the user accounts in the specified IAM groups.

Options:

* `--iam-group` (optional): Sync the user accounts on this system with the user accounts in this IAM group. At least
  one of `--iam-group` or `--iam-group-sudo` is required.
* `--iam-group-sudo` (optional): Sync the user accounts on this system with the user accounts in this IAM group and
  give these user accounts sudo privileges. At least one of `--iam-group` or `--iam-group-sudo` is required.
* `--dry-run` (optional): Print out what this command would do, but don't actually make any changes on this system.

Examples:

`ssh-iam sync-users --iam-group ssh-group --iam-group-sudo ssh-sudo-group`

#### install command

Usage: `ssh-iam install [OPTIONS]`

Description: Configure this server to a) periodically sync user accounts from IAM and b) use ssh-iam to authenticate
SSH connections.

Options:

* `--iam-group` (optional): Sync the user accounts on this system with the user accounts in this IAM group. At least
  one of `--iam-group` or `--iam-group-sudo` is required.
* `--iam-group-sudo` (optional): Sync the user accounts on this system with the user accounts in this IAM group and
  give these user accounts sudo privileges. At least one of `--iam-group` or `--iam-group-sudo` is required.
* `--cron-schedule` (optional): A cron expression that controls how often ssh-iam syncs IAM users to this system.
  Default: `*/30 * * * *` (every 30 minutes).
* `--authorized-keys-command-user` (optional): The user that should execute the SSH AuthorizedKeysCommand. Default:
  current user.
* `--dry-run` (optional): Print out what this command would do, but don't actually make any changes on this system.

## Threat model

The threat model is defined in terms of what each possible attacker can achieve.

At a high level, the security of `ssh-iam` is predicated on the security of SSH, IAM, and Gruntwork.

#### Assumptions

* Your users act reasonably and do not reveal their secrets, such as their private SSH keys or AWS credentials.
* SSH is a secure protocol and public/private key encryption guarantees are valid.
* The AWS APIs are properly secured by TLS and IAM Roles.
* The user's computer and your servers are not compromised by malware.
* The copy of `ssh-iam` running on your servers has not been compromised by malware.

#### Threats from an SSH compromise

If an attacker gets access to one of your developer's private keys, they will be able to:

* Make arbitrary changes on the server, especially if your developer had sudo access.
* Use any IAM Roles attached to that server to authenticate to the AWS APIs. In particular, the IAM Roles you need to
  use `ssh-iam` will allow the attacker to list the user names of members of any IAM group and to see the public keys
  of any of your IAM users.

Possible remediation: Mark the corresponding public key as "inactive" in the developer's IAM account, and the attacker
will no longer be able to connect to your servers.

#### Threats from an IAM compromise

If your server cannot communicate with IAM, either due to an outage in AWS or a misconfiguration on your servers:

* Your developers will not be able to authenticate to your servers using their public keys.
* User accounts will no longer automatically sync to the servers, so changes in IAM groups will not be reflected on
  your servers.

Possible remediation: Always launch servers with an [EC2 Key
Pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) as backup. This Key Pair should kept on
a trusted admin's computer, not shared with anyone else (if you must share it, make sure to encrypt it thoroughly using
something like [KMS](https://aws.amazon.com/kms/) or [Keybase](http://keybase.io/)), and only used in case of
emergencies.

If an attacker gets access to an IAM user account, they will be able to:

* Make arbitrary changes to your AWS account, depending on the permissions attached to that IAM account.
* Add their own public key to their IAM account and add that account to an IAM group that has SSH access. This will
  allow them to SSH to your servers using their public key and make arbitrary changes.

Possible remediation: Require all IAM users to use [Multi-Factor
Authentication](https://aws.amazon.com/iam/details/mfa/) and secure passwords to reduces the chances of an account
being compromised. You can even require [Multi-Factor Authentication for particularly sensitive API
calls](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html), such as any
changes in IAM or KMS. Follow the [principle of least
privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) and grant IAM users the bare-minimum permissions
they need, so if an IAM account is compromised, the attacker's abilities are limited.

#### Threats from a Gruntwork compromise

If an attacker gets access to Gruntwork's repos, they will be able to:

* Replace the `ssh-iam` binary in the [module-security repo](https://github.com/gruntwork-io/module-security-public) with one
  that contains malware. This will allow the attacker to make arbitrary changes on any server where `ssh-iam` is
  installed.

Possible remediation: Gruntwork only grants write access to its repos to employees, all of whom are required to use
Multi-Factor Authentication and strong passwords. In the future, we will publish checksums of our binaries in separate
locations so you can use those to confirm that the binary you are installing is an authentic one.

## Developing ssh-iam

Since ssh-iam makes lots of changes to the OS (creating users, modifying SSH settings, etc), we recommend running it in
a Docker container.

#### Building the Docker image

The Docker image is defined in the `Dockerfile`. To build it, run:

```
docker build -t gruntwork/ssh-iam .
```

#### Running locally

To run the app locally, we've defined a `docker-compose.yml` file that mounts source code into the Docker container
(so you don't have to keep rebuilding the container when you make source changes) and forwards our AWS environment
variables. To run the app with Docker Compose:

```
docker-compose run ssh-iam [ARGS...]
```

For example:

```
docker-compose run ssh-iam print-keys myuser
```

#### Stacktraces

If you run the app and it crashes on you, set the `SSH_IAM_DEBUG` environment variable to any value to see the
stacktrace:

```
docker-compose run -e SSH_IAM_DEBUG=true ssh-iam print-keys myuser
```

#### Running tests

**Note**: the automated tests for this repo (a) make many changes in the OS, include creating lots of OS users, so
we only recommend running them within a Docker container and (b) create and delete real IAM users and groups in your
AWS account, so always let them run to completion so they can do proper cleanup.

To run all the tests, use the `test.sh` script, which uses Docker Compose under the hood:

```
./_ci/test.sh
```

To run a specific test, use the `-run` argument:

```
./_ci/test.sh -run TestFoo
```

#### Possible future additions

Some items we are considering for the future:

1. Set up two-factor auth (2FA) using `libpam-google-authenticator`.
1. Lock down SSH settings by disabling password authentication and the root user.
1. Lock down SSH even more: https://stribika.github.io/2015/01/04/secure-secure-shell.html
1. Send notifications for all server access: https://www.inversoft.com/guides/2016-guide-to-user-data-security#intrusion-detection

If you're interested in these features, [let us know](info@gruntwork.io)!


