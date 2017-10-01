---
layout: post
title:  "Building Docker images with Travis CI"
date:   2016-08-20 16:00:00 +1200
---
I've reach the point with my level-three-rest microservice where it needs to be delivered to infrastructure which isn't my local machine.

## Iteration 0
The very basic way to this would be just to build the project locally, copy the executable and run them on the target machine. This completely defeats the purpose of having a CI process.

## Iteration 1
Lets think about this some more, we can have Travis CI build the binary, deliver it to an AWS S3 bucket which has versioning enabled. And with the appropriate AWS Auto Scaling Group and launch configuration, when the S3 bucket gets updated, you can have a lambda function scale the service out and then back in to deploy the latest binary in the S3 bucket.

This solution will work and easy to setup, however we would be consuming an entire EC2 instance to run what is currently a ~6MB binary file. What happens when you have 30 microservices deployed in this fashion? That is a lot of instances which you have to manage.

## Iteration 2
In this iteration we are going to have Travis CI build a Docker container from scratch so it should be a similar size as the compile binary file. This will be then uploaded to a Docker registry, when the EC2 instance boots it can pull the latest images and then execute the Docker run command.

## Dockerfile
The Dockerfile is very straight forward.
{% highlight js %}
FROM scratch
ADD level-three-rest /level-three-rest
CMD ["/level-three-rest"]
{% endhighlight %}

### Test Locally
Before we have Travis CI build our image for us, we should test it out locally. I do all my development in my spare time on Windows, therefore the `GOOS` flag has to be set to linux to function in Docker.
{% highlight PowerShell %}
$env:GOOS = "linux"
{% endhighlight %}

We are now ready to bulid the project and the Docker image.
{% highlight PowerShell %}
go build
docker build -t level-three-rest -f .\Dockerfile .
{% endhighlight %}

If we run `docker images` we should get the following output:
{% highlight PowerShell %}
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
level-three-rest    latest              abeffecf909d        6 days ago          6.099 MB
{% endhighlight %}

You should now be able to run the Docker container and call the API on port 8080.
{% highlight PowerShell %}
docker run -d -p 8080:8080 level-three-rest
{% endhighlight %}

## .travis.yml
The updates to the are straight forward since all we need to do is build the Docker image and upload it to our Docker hub. You can find all the information on the [Travis-CI Documents](https://docs.travis-ci.com/user/docker/).

The first thing to do is add the encrypted variables which are required to logon to Docker hub
{% highlight PowerShell %}
travis encrypt DOCKER_USER=username --add
travis encrypt DOCKER_PASS=password --add
{% endhighlight %}

Set the appropriate environment variables so the binary is statically compiled.
{% highlight YAML %}
env:
  global:
  - COMMIT=${TRAVIS_COMMIT::8}
  - REPO=bhavikk/level-three-rest
  - CGO_ENABLED=0
  - GOOS=linux
  - GOARCH=amd64  
{% endhighlight %}

Add the `docker build` step after you run `go build` in your build chain.
{% highlight YAML %}
script:
 - export TAG=`if [[ $TRAVIS_PULL_REQUEST == "false" ]] && [[ $TRAVIS_BRANCH == "master" ]]; then echo "latest"; else echo $TRAVIS_PULL_REQUEST_BRANCH; fi`
 - export REPO=bhavikk/level-three-rest
 - docker build -t level-three-rest -t $REPO:$TAG -t $REPO:$TRAVIS_BUILD_NUMBER -f Dockerfile .
{% endhighlight %}

After a successful build, we login and then upload the docker image to Docker h ub. You can set the appropriate tags before uploading the image. One other interesting thing you can perform at this stage is running the docker image and running appropriate tests against it to ensure everything passes.
{% highlight YAML %}
 after_success:
 - docker login -u $DOCKER_USER -p $DOCKER_PASS
 - docker push $REPO
{% endhighlight %}

This is a great start, we now have a public repository with our Docker image which we can pull down and run after every commit. Most companies will not want their Docker images available publically, therefore in the next blog post, I'll show how you can  push the images to AWS EC2 Container Registry from Travis-CI.
