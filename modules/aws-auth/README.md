**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/aws-auth/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# AWS Auth Helper

This module is a small wrapper script for the [AWS CLI](https://aws.amazon.com/cli/) that makes it much easier to 
authenticate to AWS when:

1. [Multi-factor authentication (MFA)](https://aws.amazon.com/premiumsupport/knowledge-center/authenticate-mfa-cli/) is
   enabled, and/or 
1. [You want to assume an IAM Role](http://docs.aws.amazon.com/cli/latest/reference/sts/assume-role.html), such as an
   IAM role that gives you access to another AWS account.





## Motivation

Normally, if MFA is enabled, setting up your credentials as environment variables is a multi-step process. First, you 
make the call to fetch the temporary STS credentials:

```bash
aws sts get-session-token --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456
```

Which returns a blob of JSON:

```json
{
  "Credentials": {
    "AccessKeyId": "AAA",
    "SecretAccessKey": "BBB",
    "SessionToken": "CCC",
    "Expiration": "DDD"
  }
}
```

Next, you have to copy and paste each part of that JSON output into the proper environment variable:

```bash
export AWS_ACCESS_KEY_ID='AAA'
export AWS_SECRET_ACCESS_KEY='BBB'
export AWS_SESSION_TOKEN='CCC'
```

If you want to assume an IAM role, you have to make another API call:

```bash
aws sts assume-role --role-arn arn:aws:iam::123456789011:role/my-role --role-session-name my-name 
```

Which returns another blob of JSON:

```json
{
  "Credentials": {
    "AccessKeyId": "EEE",
    "SecretAccessKey": "FFF",
    "SessionToken": "GGG",
    "Expiration": "HHH"
  }
}
```

Which you again have to copy into the proper environment variable:

```bash
export AWS_ACCESS_KEY_ID='EEE'
export AWS_SECRET_ACCESS_KEY='FFF'
export AWS_SESSION_TOKEN='GGG'
```

With this script, all of this can be done in a single command!




## Quick start

To install the script, you can either copy it manually to a location on your `PATH` or use the 
[gruntwork-install](https://github.com/gruntwork-io/gruntwork-installer) command:

```bash
gruntwork-install --module-name 'aws-auth' --repo 'https://github.com/gruntwork-io/module-security' --tag 'v0.4.6'
```

Now you can run the script with the exact same arguments as the AWS `get-session-token` command: 

```bash
aws-auth --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456
```

If you want to assume an IAM role, just specify the ARN of that role with the `--role-arn` parameter: 

```bash
aws-auth --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456 --role-arn arn:aws:iam::123456789011:role/my-role
```

Either way, the `aws-auth` script will write a series of `export XXX=YYY` statements to `stdout`:

```bash
aws-auth --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456

export AWS_ACCESS_KEY_ID='AAA'
export AWS_SECRET_ACCESS_KEY='BBB'
export AWS_SESSION_TOKEN='CCC'
```

To setup your AWS environment variables in one command, all you have to do is eval the result!

```bash
eval "$(aws-auth --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456)"
```




## Combining it with password managers

To be fair, using `aws-auth` isn't *really* a one-liner, since you have to set your permanent AWS credentials first:

```bash
export AWS_ACCESS_KEY_ID='<PERMANENT_ACCESS_KEY>'
export AWS_SECRET_ACCESS_KEY='<PERMANENT_SECRET_KEY>'
eval $(aws-auth --serial-number arn:aws:iam::123456789011:mfa/jondoe --token-code 123456)
```

If you store your secrets in a CLI-friendly password manager, such as [pass](https://www.passwordstore.org/), then you
can reduce this even further!

First, store your permanent AWS credentials in `pass`:

```
pass insert aws-access-key-id
Enter password for aws-access-key-id: <PERMANENT_ACCESS_KEY>

pass insert aws-secret-access-key
Enter password for aws-secret-access-key: <PERMANENT_SECRET_KEY>
```

Next, store your MFA ARN in `pass` as well:

```
pass insert aws-mfa-arn
Enter password for aws-mfa-arn: arn:aws:iam::123456789011:mfa/jondoe
```

If you will be assuming an IAM Role ARN, put that in `pass` too:

```
pass insert aws-iam-role-arn
Enter password for aws-iam-role-arn: arn:aws:iam::123456789011:role/my-role
```

Now, you can store a script in `pass` that ties all of this together. Run the `insert` command with the `-m` parameter
so you can enter multiple lines:

```
pass insert -m aws-sts-env-vars
Enter contents of aws-sts-env-vars and press Ctrl+D when finished:
```

And now enter the following script:

```bash
read -p "Enter your MFA token: " token
eval $(AWS_ACCESS_KEY_ID=$(pass aws-access-key-id) AWS_SECRET_ACCESS_KEY=$(pass aws-secret-access-key) aws-auth --serial-number $(pass aws-mfa-arn) --token-code "$token")
```

If you want the script to assume an IAM role, just add the `--iam-role` parameter at the end:

```bash
read -p "Enter your MFA token: " token
eval $(AWS_ACCESS_KEY_ID=$(pass aws-access-key-id) AWS_SECRET_ACCESS_KEY=$(pass aws-secret-access-key) aws-auth --serial-number $(pass aws-mfa-arn) --token-code "$token" --role-arn $(pass aws-iam-role-arn))
```

Now, to setup your temporary STS credentials is *truly* a one-liner!
 
```bash
eval "$(pass aws-sts-env-vars)"
``` 

*Note: the double quotes around the `$()` are required.*


