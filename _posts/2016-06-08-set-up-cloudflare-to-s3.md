---
layout: post
title:  "Setup Cloud Flare to S3 Bucket"
date:   2016-06-08 05:00:00 +1200
---
This post is going to assume you have no records for the site you want to front with Cloudflare. You have to ensure that your S3 Bucket has the same name as the alias in the CNAME. E.g: If you used `files` then your S3 bucket name must be called `files.mydomain.tld`

Check out my [GitHub](https://github.com/bhavikkumar/cloudformation-templates) repository for a template to create S3 buckets which are ready to be served as a public website.

## CloudFlare Setup
The first step is to click on "Add Website" and type in yourdomain.tld and then begin the scan. This usually takes about a minute. Once it is complete click on "Continue Setup".

The second step is to setup your DNS records.
![CloudFlare DNS Setup]({{ site.baseurl }}/assets/cloudflare_screen_1.png)

The third step is to select a CloudFlare plan, I went with the Free Website since the additional features are not required for my blog.

The fourth step is to go to your domain registrars website and configure the nameservers to be the ones which CloudFlare have assigned you. The Cloudflare have a [knowledge base](https://support.cloudflare.com/hc/en-us/articles/206455647-How-do-I-change-my-domain-nameservers-) for the popular registrars.

The next things to setup will depend on your needs. I setup DNSSEC, HSTS and <s>SSL</s>TLS for this blog.