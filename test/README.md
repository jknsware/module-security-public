**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/test/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Tests

This folder contains the tests for the modules in this repo.

## Running the tests locally

**Note #1**: Many of these tests create real resources in an AWS account. That means they cost money to run, especially
if you don't clean up after yourself. Please be considerate of the resources you create and take extra care to clean
everything up when you're done!

**Note #2**: Never hit `CTRL + C` or cancel a build once tests are running or the cleanup tasks won't run!

**Note #3**: We set `-timeout 45m` on all tests not because they necessarily take 45 minutes, but because Go has a
default test timeout of 10 minutes, after which it does a `SIGQUIT`, preventing the tests from properly cleaning up
after themselves. Therefore, we set a timeout of 45 minutes to make sure all tests have enough time to finish and
cleanup.

#### Prerequisites

- Install the latest version of [Go](https://golang.org/).
- Install [glide](https://glide.sh/) for Go dependency management. On OSX, the simplest way to install is
  `brew update; brew install glide`.
- Install [Terraform](https://www.terraform.io/downloads.html).
- Add your AWS credentials as environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- For some of the tests, you also need to set the `GITHUB_OAUTH_TOKEN` environment variable to a valid GitHub
  auth token with "repo" access. You can generate one here: https://github.com/settings/tokens

#### Setup

Download Go dependencies using glide:

```
cd test
glide update
```

#### Run all the tests

```bash
cd test
go test -v -timeout 45m -parallel 128
```

**Note**: The automated test for the `kms-master-key` package is disabled by default. That's because generating a KMS
Master Key costs $1/month, even if we delete it right after, which can add up quickly if we run this test often. To
enable the test, you need to set the `RUN_KMS_TEST` environment variable:

```bash
cd test
RUN_KMS_TEST=true go test -v -timeout 45m -parallel 128
```

#### Run a specific test

To run a specific test called `TestFoo`:

```bash
cd test
go test -v -timeout 45m -parallel 128 -run TestFoo
```