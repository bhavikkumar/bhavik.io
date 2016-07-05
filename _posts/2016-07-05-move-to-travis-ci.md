---
layout: post
title:  "I dropped Jenkins for Travis CI"
date:   2016-07-05 20:00:00 +1200
---
In a previous post I was using Jenkins in my CI/CD workflow, however in the last week, I've decided to move to Travis-CI due to some issues with Jenkins.

The main challenges I ran in to was how to backup of Jenkins configuration and the configuration being separate from the actual code. The former is fairly important especially if you are running the environment inside AWS. Jenkins was running inside a container on a t2.micro instance, either the container or the EC2 instance could have a failure.

There are solutions for this problem which are:
 - Use the SCM Sync Plugin to Github for configuration - then the question about secrets arose.
 - Or have the EC2 instance in a auto scaling group with a launch configuration which attaches a EBS which gets a snapshot taken daily or use Amazon EFS if it is available in the region. And when the docker image starts, it would have the correct data volume added to it. This is not bad but it also costs money, and the cost will grow with the number of jobs.

I've been thinking more about CI/CD to make life easier when it comes to operational tasks, this is how the latter problem arose regarding the CI/CD configuration. I like the idea of having everything that is required in the same repository as the source code, this includes tests, CI/CD pipeline and anything else which is required. I feel Jenkins takes me away from this while Travis-CI is a simple Yaml file next to my source code.

I can hear some of you saying, wait wasn't the reason for using Jenkins was for private repositories. Yes, you are right, it was. However, I cannot see myself having any private git repositories in the foreseeable future. The only time I would need a private git repository was if I was building a product or service to sell some time in the future and in this scenario I am sure that it would be possible to afford for GitHub and a service such as Travis-CI or Buildkite.

_Failure should be our teacher, not our undertaker. Failure is delay, not defeat. It is a temporary detour, not a dead end. Failure is something we can avoid only by saying nothing, doing nothing, and being nothing._ - Denis Waitley
