<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/ftp/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# AWS Transfer FTP Server

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions an AWS SFTP server. It allows you to manage the FTP server and its users wtih code. You can deploy an FTP server with consistent settings on multiple environments with the same infrastructure code.

* Can create and manage an S3 Bucket or use an existing S3 bucket as the storage layer for the FTP server.
* Can create and manage ftp users with code using the `@ftp_users` variables.  You can use different environment configs if you want different users on production and development environments. Of course, you can also just add users manually deployment also. You have the flexiblity to choose.
* Can create an connect a route53 record to ftp server automatically. Though, please read the Hostname section for pros and cons.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/ftp values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "ftp", git: "git@github.com:boltopspro/ftp.git"
```

## Configure

First, you want to configure the [configs](https://lono.cloud/docs/core/configs/) files. Use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed ftp

To deploy to additional environments:

    LONO_ENV=production  lono seed ftp

The generated files in `config/ftp` folder look something like this:

    configs/ftp/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Here's an example of the configs:

    # Parameter Group: AWS::S3::Bucket
    # BucketName=
    # ExistingBucket=

    # Parameter Group: AWS::Transfer::Server
    # EndpointType=PUBLIC
    # IdentityProviderType=SERVICE_MANAGED

    # Parameter Group: AWS::Route53::RecordSet
    # ConnectToDns=public # public # public or private
    # DnsName= # ftp.example.com.
    # DnsTtl=60
    # DnsType=CNAME
    # HostedZoneId= # /hostedzone/Z01963071SQH02EXAMPLE
    # HostedZoneName= # example.com.

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy ftp --sure
    LONO_ENV=production  lono cfn deploy ftp --sure

## Configure Details

### Users

You can configure users for your ftp server with the `@ftp_users` variable.  Example:

```ruby
@ftp_users = [{
  UserName: "testuer",
  SshPublicKeys: ["ssh-rsa AAAAB3..ABC== testuer"],
}]
```

### Hostname

This blueprint can create a route53 record and automatically connect it to the FTP hostname. However, there are some pros and cons of this.

* Pro: It's automated
* Con: It will not show up on the SFTP console as the Hostname, which is very friendly.

This is because when you connect the Hostname with the SFTP console, AWS does it a little differently.  Essentially, AWS tags the FTP server with a special tag, and that in turn updates a route53 record internally. Only AWS is permitted to use the special tags. AWS uses this tag info to show the Hostname in the Servers list view.

So if you want the Hostname to show up on the SFTP console, do not configure any of the `AWS::Route53::RecordSet` settings when you deploy the stack. Instead, manually connect up the Hostname afterwards in the AWS Transfer console.
