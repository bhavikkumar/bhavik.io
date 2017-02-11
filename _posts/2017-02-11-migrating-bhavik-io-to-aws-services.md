---
layout: post
title:  "Migrating bhavik.io to AWS CloudFront"
date:   2017-02-09 16:00:00 +1200
---
At AWS Re:invent it was annouced that CloudFront and other AWS services now have AWS Shield which provides DDoS protection. This was the main reason I have been using CloudFlare. With AWS Shield and CloudFront, I thought it would be a good time to migrate [bhavik.io](https://bhavik.io) to use AWS services, this means using Route 53, S3 and CloudFront. I will also be taking this time to migrate everything in to the US-East-1 region.

## Migrate S3 Bucket to US-East-1
Unfortunately, there was no way to do this without an outage while using CloudFlare. The reason being is deleting a bucket and recreating it in another region isn't guaranteed by AWS. The following is the error message which you will receive when trying to recreate the bucket straight after deleting it.
{% highlight shell %}
An error occurred (OperationAborted) when calling the CreateBucket operation: A conflicting conditional operation is currently in progress against this resource. Please try again.
{% endhighlight %}

It took about 30 minutes for my bucket name to become available again but it could take hours before becoming available again, if at all. AWS do not recommend removing production buckets at all as you may lose the bucket name if you are unlucky.

## CloudFormation
Creating the previous CloudFormation templates, I learnt that I should simplify my stacks. I created very fine grained stacks previously. This time I'm going to create a 'product' stack where it makes sense and use tags (where possible) to identify which AWS resources belong to which 'product'. I started by deleting all of the existing CloudFormation stacks which were created for this blog.

### DNS
The first stack is for Route 53 hosted zone, I decided to create individual stacks for each hosted zone since each domain could be used for different purposes and not just my blog. It also makes it easy to delete a individual hosted zone when no other CloudFormation stack references the Route 53 stack for the hosted zone id.

Once all the DNS entries were populated, I updated all the nameservers on my namecheap account to point to the nameservers which were listed under Route 53.

## Website
The second template creates a static website stack with CloudFront CDN. This stack fixes some flaws of my original deployment such as not having any logging turned on. The basis of this template came from [here](https://alestic.com/2016/10/aws-git-backed-static-website/). I removed the items which were not required and changed some others.
 - S3 Bucket
 - Build User and IAM Policy
 - SSL Certificate
 - CloudFront Distribution
 - Access Logs to S3 Bucket
 - Route 53 DNS Record
 - Cloudwatch Alarms (4xx and 5xx Error Rates)

When I first launched the stack, I thought that the stack got stuck and failed, but actually I had did not realise that I got sent a approval email for the certificate which was issued by Amazon Certificate Manager. The stack takes a long time to deploy once, the CloudFront distribution is being deployed, so it is best to go and do something else before checking on the status or have a script which notifies you when it is complete. I've just deployed to the US, Canada and Europe edges and it took roughly 60 minutes for the stack creation to complete.

## E-Mail
Originally I was planning on using SES for my mail forwarding. This would of meant having extra complexity as forwarding with SES requires S3 and Lambda function. I decided in the end to go with the simplicity of mailgun as it only requires DNS entries. However, if you are interested in setting up fowarding with SES, then check out the following repoistory https://github.com/arithmetric/aws-lambda-ses-forwarder.

## Closing Thoughts
The ability to export variables and the fact now CloudFormation templates are YAML has definitely made it easier to write templates. I'm looking forward to using cross-stack references a lot more. The only thing I wish for is the ability to use `!Ref` in the ImportValue and Export fields.

All the templates I created or used can be found on [GitHub](https://github.com/bhavikkumar/cloudformation-templates)
