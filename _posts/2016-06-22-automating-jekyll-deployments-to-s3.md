---
layout: post
title:  "Automating Jekyll Deployments to S3"
date:   2016-06-22 05:00:00 +1200
---
This post is going to carry on from the series of Jekyll posts, in this post I'll be explaining how to automatically deploy to your S3 bucket using Jenkins. I'll be assuming that your Jekyll blog is being uploaded to a Git repository.

The reason I've picked Jenkins instead of something like Travis CI/Circle CI is that I can work around any restrictions which I may have in the future, such as having private repos.

To make this easier, I've actually created a docker container which contains Jenkins and Jekyll already to make it easier to get started. It can be found on [docker hub](https://hub.docker.com/r/bhavikk/jenkins-jekyll/) with the instructions to run it.

### AWS Security
Jenkins will need access to the S3 bucket, I would recommend creating a specific build user with the minimum IAM policies required. You will have to have a access and secret keys created for the build user. I've got a build user cloudformation template which can be found on [GitHub](https://github.com/bhavikkumar/cloudformation-templates/blob/master/bk-jenkins-build-user.template). Note that this template may have additional policies for my own needs.

### Jenkins Plugins
The following plugins will need to be configured:

- Git Plugin
- S3 Publisher Plugin 0.8

#### S3 Publisher Plugin Configuration
The reason for using version 0.8 of the S3 publisher plugin is I've had issues with the latest version (0.12 at the time of this post).  To configure the plugin click on "Manage Jenkins" -> "Configure System", then scroll down to the Amazon S3 profiles section and fill in the details as shown in the following screenshot.

![Jenkins AWS S3 profile]({{ site.baseurl }}/assets/jenkins-s3-publisher.png)

### Jenkins Job Configuration

The job configuration is pretty straight forward. Under the Source Code Management ensure you have set the correct Git Repository URL so that the source can be pulled down.

I have my personal blog setup to build periodically every day, however you can trigger it on check in if required. The next part is the most important as in the build step you will want to execute a shell command as shown in the screenshot.

![Jenkins Jekyll Build Configuration]({{ site.baseurl }}/assets/jenkins-jekyll-build-config.png)

The final step is to have the S3 Publisher Plugin to push to the appropriate S3 bucket as part of the post build actions.

![Jenkins Jekyll Build Configuration]({{ site.baseurl }}/assets/jenkins-jekyll-post-build-config.png)

Once everything is configured, it shoudl be a simple task of hitting the build button to ensure that everything works properly.