---
layout: post
title:  "AWS Billing and Budgets"
date:   2017-02-15 16:00:00 +1200
---
Now that my free tier has expired, I didn't want to get bill shock when AWS finally bills me for the services I'm using. Therefore I decided to have a look at the billing and cost management tools which have been implemented in AWS in recent times.

In my case I just wanted a simple notification of usage when certain billing tags go over a certain usage threshold, this then allows me to manually adjust things if required. In the future I may implement a Lambda function which automatically helps with keeping the forecasted costs down. If you want to trigger things off budget alerts then the first thing to do is setup a SNS topic if you don't already have one, I have a template for global resources, my billing topic is inside this [global template](https://github.com/bhavikkumar/cloudformation-templates/blob/master/global.yaml).

To create a budget you can use the CLI or the console. Unfortunately, I haven't seen a way of doing this using CloudFormation, but it would be great if it was possible, then the budgets could be part of the stacks being deployed. The way I did the alerts is by creating a AWS Budget using the console and then pointing it at my billing topic. The screenshot below shows you the configuration I used for bhavik.io based on the bill-to tag I have on my resources, now I won't be in for a nasty surprise when I get my AWS bill.

![bhavik.io budget]({{ site.baseurl }}/assets/aws-bhavik.io-budget.png)
