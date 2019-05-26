---
description: Amazon Web Services
---

# AWS

## S3 Bucket Visible

```text
site:s3.amazonaws.com
site:blob.core.windows.net  (For Azure)

```

## S3 Excessive Permissions

Ocassionally S3 buckets are configured with incorrect permissions. In the instance of the below example, the bucket is visible online but has been set with the List permission globally. Consequently, anyone can list the bucket contents.

```bash
➜  ~ aws s3 ls s3://flaws.cloud --no-sign-request
2017-03-14 03:00:38       2575 hint1.html
2017-03-03 04:05:17       1707 hint2.html
2017-03-03 04:05:11       1101 hint3.html
2018-07-10 17:47:16       3082 index.html
2018-07-10 17:47:16      15979 logo.png
2017-02-27 01:59:28         46 robots.txt
2017-02-27 01:59:30       1051 secret-dd02c7c.html
```

In other instances, ocassionally a bucket will be set to "Any Authenticated AWS User". In this instance, the bucket can be listed, but an AWS account must be created. In this instance, a profile has been created.

```bash
➜  ~ aws configure --profile reader
AWS Access Key ID [None]: ****
AWS Secret Access Key [None]: ****
Default region name [None]:
Default output format [None]:
```

The listing is shown with and without this profile applied.

```bash
➜  ~ aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/ --no-sign-request

An error occurred (AccessDenied) when calling the ListObjectsV2 operation: Access Denied
➜  ~ aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/ --profile reader
2017-02-27 02:02:15      80751 everyone.png
2017-03-03 03:47:17       1433 hint1.html
2017-02-27 02:04:39       1035 hint2.html
2017-02-27 02:02:14       2786 index.html
2017-02-27 02:02:14         26 robots.txt
2017-02-27 02:02:15       1051 secret-e4443fc.html
```

In these buckets you may find useful files that the developer left, ones unintended to be seen by all. To download a bucket use the `aws s3 sync` command.

#### Examples

* [https://hackerone.com/reports/229690](https://hackerone.com/reports/229690)



## EC2 Snapshots

EC2 Snapshots save the state of an EC2 instance. Ocassionally these can end up being set to public.

## Cloud Metadata

As described by flaws.cloud:

> The IP address 169.254.169.254 is a magic IP in the cloud world. AWS, Azure, Google, DigitalOcean and others use this to allow cloud resources to find out metadata about themselves. Some, such as Google, have additional constraints on the requests, such as requiring it to use `Metadata-Flavor: Google` as an HTTP header and refusing requests with an `X-Forwarded-For` header. AWS has no constraints. If you can make any sort of HTTP request from an EC2 to that IP, you'll likely get back information the owner would prefer you not see.

Suddenly, a SSRF can be extended to be far more dangerous.

#### APIs

* [Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service#metadata-apis)
* [Google Cloud Platform](https://cloud.google.com/compute/docs/storing-retrieving-metadata)
* [Amazon Web Services](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

#### Examples

* [https://hackerone.com/reports/401136](https://hackerone.com/reports/401136)
* [https://hackerone.com/reports/53088](https://hackerone.com/reports/53088)
* [https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2017/august/when-a-web-application-ssrf-causes-the-cloud-to-rain-credentials-and-more/](https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2017/august/when-a-web-application-ssrf-causes-the-cloud-to-rain-credentials-and-more/)
* 
## Learning Environments

{% embed url="http://flaws.cloud" %}

{% embed url="https://rhinosecuritylabs.com/aws/cloudgoat-vulnerable-design-aws-environment/" %}





