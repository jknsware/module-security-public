**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/fail2ban/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Fail2Ban Module Example

This is an example of how to use the [fail2ban module](/modules/fail2ban) to configure a Linux server to
automatically ban malicious ip addresses. The example contains a [Packer](https://www.packer.io/) template that creates
either an Ubuntu or Amazon Linux AMI and installs and configures `fail2ban` on them.

## Quick start

To build the AMIs:

1. Install [Packer](https://www.packer.io/)
1. Set your [GitHub access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/) as
   the environment variable `GITHUB_OAUTH_TOKEN`.
1. Run `packer build fail2ban-amazon-example.json` or `packer build fail2ban-ubuntu-example.json`