**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/examples/cloudtrail/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# CloudTrail Example

This is an example of how to use the [cloudtrail module](/modules/cloudtrail) to enable AWS CloudTrail on your AWS account.

This example will create an S3 Bucket where CloudTrail events can be stored, a KMS Customer Master Key (CMK) used to 
encrypt CloudTrail events, and a CloudTrail "Trail" itself to enable API events to be recorded and stored in S3.

See the [cloudtrail module](/modules/cloudtrail) for additional details.

## Quick start

To try these templates out you must have Terraform installed:

1. Open `vars.tf` and fill in any variables that don't have a default. 
1. Run `terraform plan` to confirm that Terraform will create what looks like a reasonable set of resources.
1. Run `terraform apply` to enable CloudTrail.
1. Now log into the AWS Web Console, go to the CloudTrail service, and within a few minutes you should see records of 
   API events available for browsing.