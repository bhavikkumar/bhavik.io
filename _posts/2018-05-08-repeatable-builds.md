---
layout: post
title:  "Versioning and Repeatable Builds"
date:   2018-05-12 10:00:00 +1200
---
This post is regarding my views which make trade offs to reduce production risk by making lives of engineers slightly harder. As someone who is on call, I prefer knowing exactly what is running in production so that I can easily support, troubleshoot and resolve issues quickly.

# Versioning
All engineers have opinions around versioning and what to version. Some don't version anything, others version everything and some pick something in between. My opinion is that anything which will be used in or against production systems needs to be versioned.

I personally am a fan of using [semantic versioning](https://semver.org/) but any versioning scheme is fine as long as you can identify what is running in your production environment easily.

# Dependencies
Dealing with dependencies is the most important step of being able to have repeatable builds. If dependencies aren't handled correctly you will lose the ability to ensure that your production builds are exactly the same in the unfortunate scenario you lose arefacts which are being deployed.

The biggest detriment to repeatable builds is the use of ranges for dependencies. For example in Maven having the following [1.0.0,2.0.0) allows versions 1.0.0 <= x < 2.0.0. This helps engineers to ensure they are getting the latest version of the dependencies when they build, however they lose the repeatability of builds. Some engineers argue this can be dealt with using plugins with Maven etc, but this is adding extra complexity. Others say nothing bad will happen, this is not true, you cannot guard against a engineer (especially external to the team) accidently putting in a breaking change.

I am of the opinion regardless of if you are building a monolith or microservices that dependency version have to be explicit. Dealing with dependencies in such a manner does mean engineers/teams have some extra overhead with having to move dependencies to newer versions but I think this is a better option than trying to figure out why the build is broken at 3AM while trying to resolve a production issue.

There are multiple ways teams can deal with this overhead and I would leave it up to each team on how they wawnt to deal with this problem. My preference is that if a build has not occurred in over 30 days or before adding a feature that the dependencies are checked and upgraded.

# Wrap Up
To achieve repeatable builds it is important to handle versioning and dependencies correctly.
* Use a verion number scheme, especially if third parties (this could be another team in the same organisation) will be using the component being built.
* Version all artefacts which will be released or used in production.
* Do not use ranges in dependency version numbers.
* Build a process around handling dependency upgrades
