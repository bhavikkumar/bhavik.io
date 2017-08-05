---
layout: post
title:  "AWS Stack Sets"
date:   2017-08-05 16:00:00 +1200
published: false
---
I've been doing a lot of work with AWS Organisations recently and there was a large amount of repetition across the accounts. This has now been resolved by AWS with the release of AWS Stack Sets. AWS already have good [documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs.html) regarding stack sets and best practices. This post is going to be on how I simplified my deployment across accounts using stack sets.

The model I have used for my organisation setup is having a master account where there is a few admin user accounts. The majority of the accounts are in an identity account which does not allow users to do anything except assume roles in to the various other accounts. For my examples the stack sets have to be applied from the master account in to the other accounts to have standardisation across all the other accounts.

## Prerequisites
The first thing we have to do is setup a trust between the accounts as per the prerequisite steps in the documentation. The first step is to allow CloudFormation in the master account to assume the AWSCloudFormationStackSetExecutionRole. Both of the templates below can be found on the AWS Stack Sets documentation page above.

## CloudTrail
The first and most common use case will be deploying CloudTrail across all accounts and centralising the logs to the master account. There is a sample template already available, the downside of the sample template is that it does not use KMS for encryption. The template can be found [here](https://github.com/bhavikkumar/cloudformation-templates/blob/master/cloudtrail.yaml), modify it depending on the number of accounts you have and run the template in the master account first. The template will output the KMS Key ARN and the S3 Bucket which will be used for the input of the stack set.

Run the template as a stack set, provide stack name, S3 bucket and KMS Key ARN as the input parameters and click next. The next screen is the Set Deployment Options, here we are going to use the "Deploy stacks in accounts" and provide the list of all the accounts we want to execute this template in. Then we are going to specify the region which we are going to run the template in, since this is a global CloudTrail we only have to specify a single region.

For the deployment options, I left it as the default settings, however if you want to deploy faster then you can increase the maximum concurrent accounts.

## IAM Roles
The next set of stacks I created were the IAM roles which are required by users in the identity account so they can actually perform their functions. The nice thing here is because stack sets take a list of accounts, I can just omit the identity account from receiving the roles.

## Conclusion
I think AWS CloudFormation Stack Sets definitely makes it simiplier to deploy common changes across accounts and regions. One thing which needs work on is feedback to the user, I wish it would provide what account and region is being modified.
