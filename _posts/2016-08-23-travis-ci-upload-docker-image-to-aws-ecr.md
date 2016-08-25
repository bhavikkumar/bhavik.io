---
layout: post
title:  "Upload Docker Image to AWS ECR using Travis CI"
date:   2016-08-23 10:00:00 +1200
---
In my last post I showed how to upload a Docker image to Docker Hub. For AWS users, you might find it cheaper using AWS ECR compared to private Docker Hub repositories, especially if you are creating images using statically compiled Go binaries.

For example, lets say you have some "huge" Go binaries of 40MB, then you could store 50 Docker images for USD$0.20/month using AWS ECR, on Docker Hub that would cost you $50/month.

## .travis.yml
This is the only file which requires modifications from the last time. You will need to encrypt your AWS Access Key and AWS Secret Access Key. I am also going to encrypt my AWS Account Number because it seems like the right thing to do, but it will be printed out when running docker push.
{% highlight PowerShell %}
travis encrypt AWS_ACCESS_KEY_ID=MYACCESSKEY --add
travis encrypt AWS_SECRET_ACCESS_KEY=MYSECRETACCESSKEY --add
travis encrypt AWS_ACCOUNT_NUMBER=MYACCOUNTNUMBER --add
{% endhighlight %}

The build step will continue to contain the Docker build command as previously outlined.
{% highlight YAML %}
- docker build -t level-three-rest -t $REPO:$TRAVIS_BUILD_NUMBER -f Dockerfile .
{% endhighlight %}

The last thing to change is the `after_success` section to the following, you can change the tags to your preferred style.
{% highlight YAML %}
after_success:
- pip install --user awscli
- export PATH=$PATH:/$HOME/.local/bin
- aws ecr create-repository --repository-name $REPO --region us-west-2
- eval $(aws ecr get-login --region us-west-2)
- docker tag $REPO:$TRAVIS_BUILD_NUMBER $AWS_ACCOUNT_NUMBER.dkr.ecr.us-west-2.amazonaws.com/$REPO:$TRAVIS_BUILD_NUMBER
- docker push $AWS_ACCOUNT_NUMBER.dkr.ecr.us-west-2.amazonaws.com/$REPO:$TRAVIS_BUILD_NUMBER
{% endhighlight %}

I ran in to a issue where I had to create the repository before I could push Docker images to it, that is why in the above examples, I've added the following command `aws ecr create-repository --repository-name $REPO --region us-west-2`.

If you navigate to the AWS console you should have the following:
![AWS ECR Docker Image]({{ site.baseurl }}/assets/aws-ecr-ltr-docker-images.png)

## Permissions to AWS ECR
The following policy will grant the Travis-CI build user to create AWS ECR repositories if they do not exist, as well as push images.
{% highlight JSON %}
"Action": [
  "ecr:CompleteLayerUpload",
  "ecr:CreateRepository",
  "ecr:GetAuthorizationToken",
  "ecr:GetDownloadUrlForLayer",
  "ecr:InitiateLayerUpload",
  "ecr:UploadLayerPart"
],
"Effect": "Allow",
"Resource": "*"
{% endhighlight %}
