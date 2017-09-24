**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/ntp/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# NTP Module

This module installs and configures [NTP](http://www.ntp.org/) on a Linux server. This helps prevent clock drift on the
server; if the clock drifts too far, you'll get all sorts of strange errors, such as AWS API calls failing due to 
invalid timestamps.

This script currently works on:

* Ubuntu
* Amazon Linux (currently a no-op, since Amazon Linux already has NTP installed & configured)




## How do you use this module?

The best way to use this module is to run the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer) 
in a [Packer](https://www.packer.io/) template:

```bash
gruntwork-install --module-name ntp --tag v0.6.1 --repo https://github.com/gruntwork-io/module-security
```

See the [ntp example](/examples/ntp) for working sample code.
