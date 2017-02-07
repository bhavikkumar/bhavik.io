---
layout: post
title:  "Migrating to bhavik.io to AWS Services"
date:   2016-12-23 16:00:00 +1200
---
At AWS Re:invent it was annouced that CloudFront and other AWS services now have AWS Shield which provides DDoS protection. This was the main reason I have been using CloudFlare. With AWS Shield and CloudFront, I thought it would be a good time to migrate [bhavik.io](https://bhavik.io) to only use AWS services, this means using Route 53, S3, CloudFront and SES. I will also be taking this time to migrate everything in to the US-East-1 region.

## Migrate S3 Bucket to US-East-1
I ensured that I had no posts which were set to be published automatically in the future and then manually migrated the S3 bucket using the AWS CLI and console.

I created `tempsite-bhavik.io` bucket in the US Standard region. Once this was completed I enabled website hosting manually.
{% highlight shell %}
aws s3api create-bucket --acl public-read --bucket tempsite-bhavik.io --region us-east-1
{% endhighlight %}

I then ran the AWS S3 Sync command from the CLI.
{% highlight shell %}
aws s3 sync s3://site-bhavik.io s3://tempsite-bhavik.io --source-region us-west-2 --region us-east-1
{% endhighlight %}

I ensured that I could access the site and updated the CloudFlare DNS to point to the new site. Next was to delete the existing CloudFormation stacks. Once I had the blog in US-East-1 region, the next step was to actually start creating the CloudFormation templates to setup the stacks correctly.

## CloudFormation
Creating the previous CloudFormation templates, I learnt that I should simplify my CloudFormation stack. I created very fine grained stacks previously. This time I'm going to create a 'product' stack where it makes sense and use tags (where possible) to identify which AWS resources belong to which 'product'.

### DNS
The first stack is for Route 53 hosted zone, I decided to create individual stacks for each hosted zone since each domain could be used for different purposes and not just my blog. It also makes it easy to delete a individual hosted zone when no other CloudFormation stack references the Route 53 stack for the hosted zone id.

{% highlight yaml %}
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Route 53 Hosted Zone'
Metadata:
  'AWS::CloudFormation::Interface':
    ParameterGroups:
    - Label:
        default: 'Hosted Zone Parameters'
      Parameters:
      - DomainName
Parameters:
  DomainName:
    Description: 'The Domain Name'
    Type: String
    Default: ''
Conditions:
  HasName: !Not [!Equals [!Ref DomainName, '']]
Resources:
  bhavikIoDNS:
    Type: 'AWS::Route53::HostedZone'
    Properties:
      HostedZoneConfig:
        Comment: !Join [ '', [ 'Hosted Zone for', !Ref DomainName ] ]
      Name: !Ref DomainName
      HostedZoneTags:
      -
        Key: 'Domain'
        Value: !Ref DomainName
Outputs:
  ZoneId:
    Value: !Ref bhavikIoDNS
    Description: 'Hosted Zone Id'
    Export:
      Name: !Sub '${DomainName}-hosted-zone-id'
{% endhighlight %}


## Website
The scond template creates a static website stack with CloudFront CDN. This stack fixes some flaws of my original deployment such as not having any logging turned on. This template created the following:
 - S3 Bucket
 - Build User
 - SSL Certificate
 - CloudFront Distribution
 - Logging to S3 Bucket
 - Route 53 DNS Record (In the appropriate Hosted Zone)
 - Cloudwatch Alarms (4xx and 5xx Error Rates)

{% highlight yaml %}

{% endhighlight %}

## E-Mail



## Costs
It has been over 12 months since I first registered my AWS account and therefore the free tier has expired which means controlling costs is important. I expect this blog to only cost ~USD$0.65/month. What I have done is setup a billing alert and also using my tags I can easily create billing report and see which 'product' is costing the most.

## Thoughts
The updates to CloudFormation has definitely made it easier to write templates, the cross stack references have definitely helped.
