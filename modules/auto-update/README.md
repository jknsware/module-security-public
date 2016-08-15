**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/auto-update/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Auto Update Module

This module can configure a Linux server to automatically install security updates. This module currently supports
Ubuntu 14.04/16.04 (using [unattended-upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)) and
Amazon Linux (using [yum-cron](http://man7.org/linux/man-pages/man8/yum-cron.8.html)).

## How do you use this module?

#### Example

See the [auto-update example](/examples/auto-update) for an example of how to use this module.

#### Installation

To use this module, you just need to install and run the `configure-auto-update` script on your servers. The best way
to do that is to use the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer) in a
[Packer](https://www.packer.io/) template:

```
gruntwork-install --module-name auto-update --tag v0.0.2 --repo https://github.com/gruntwork-io/module-security
```

#### Ubuntu support

On Ubuntu, we use [unattended-upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html) to
automatically install updates. A cron job runs `unattended-upgrades` once per day. Our default configuration is as
follows:

* Run `unattended-upgrades` daily via a cron job, with a random delay between 0 and 30 minutes (configured via the
  `APT::Periodic::RandomSleep`) so all of your servers don't update at the exact same time.
* On each run of `unattended-upgrades`, update the package list, download updates, and install updates. Once per week,
  clean up the local download archive.
* Only use updates from the Debian Security Updates channel, which should contain security updates prepared by the
  Debian Security Team and/or by package maintainers. Check out [security.debian.org](http://security.debian.org) for
  what the types of updates are included on this channel. If you use custom APT repos, you may need to add other
  origins or channels (see [Repositories for Stable
  Users](https://debian-handbook.info/browse/stable/apt.html#idm140485892366240)) using a custom configuration file,
  as specified below.
* Log all `unattended-upgrades` output `/var/log/unattended-upgrades`.

You can find most of this configuration under `install-scripts/unattended_upgrades_config.txt`. If you wish to specify
a custom configuration, you can use the `--unattended-upgrades-config` option when installing the `auto-update` module
(if you're using `gruntwork-install`, you'll need to use the `--module-param` option, such as
`gruntwork-install --module auto-update --module-param unattended-upgrades-config=/foo/bar/my_unattended_upgrades_config`).

#### Amazon Linux Support

On Amazon Linux, we use [yum-cron](http://man7.org/linux/man-pages/man8/yum-cron.8.html) and
[yum-security](http://linux.die.net/man/8/yum-security) to automatically install updates. Note that Amazon Linux
automatically installs [security updates at launch
time](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonLinuxAMIBasics.html#security-updates), but if you don't
redeploy your servers very often (e.g. the EC2 Instances in an ECS Cluster), your server may go a long time without
security updates, so we use `yum-cron` to update the server more often. Our default configuration is as follows:

* Run `yum-cron` daily via a cron job, with a random delay between 0 and 30 minutes (configured via the
  `random_sleep` variable) so all of your servers don't update at the exact same time.
* On each run of `yum-cron`, update the package list, download updates, and install updates.
* On each run of `yum-cron`, run `yum --security upgrade`. This will install all updates marked as security updates in
  your yum repos. By default, Amazon Linux comes with the `amzn-main` and `amzn-updates` yum repos and you can find a
  list of Amazon security bulletins [here](https://alas.aws.amazon.com/).
* Log all `yum-cron` output to `/var/log/cron` and `/var/log/yum.log`

You can find most of this configuration under `install-scripts/yum_cron_config.txt`. If you wish to specify a custom
configuration, you can use the `--yum-cron-config` option when installing the `auto-update` module (if you're using
`gruntwork-install`, you'll need to use the `--module-param` option, such as
`gruntwork-install --module auto-update --module-param yum-cron-config=/foo/bar/my_yum_cron_config`).

## Limitations

* This module only updates software installed on your servers using the default package manager (`apt` for Ubuntu,
  `yum` for Amazon Linux). Any software you install through other means, such as downloading binaries through `curl` or
  using an alternate package manager (e.g. `gem`, `npm`, `nix`, etc) will NOT be updated.
* The default configuration only installs security updates. Other types of bug fixes, even serious, but not-security
  sensitive ones, will NOT be installed automatically.
* The default configuration of this module does NOT reboot your servers automatically. This helps avoid unexpected
  downtime, but it means that any security updates that require a reboot to work will require you to update your AMIs
  and redeploy them yourself.


## TODO

* Both `unattended-upgrades` and `yum-cron` support email notification after each update. We could enable this,
  possibly using Amazon SES to send the emails.

