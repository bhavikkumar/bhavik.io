---
layout: post
title:  "Travis CI with Coveralls"
date:   2016-08-15 18:15:00 +1200
---
I've been learning Go and eventually build a set of microservices which can meet the third level in the rest maturity model. This project is going to the basis of a number of up coming posts.

One of the items which I wanted to have from the start was to have constant feedback on builds, tests and coverage. Once I had automated build process working, the next thing was to find a service which allowed feedback on code coverage. After a bit of research the options, I came to [coveralls](https://coveralls.io) since it supports a large number of languages.

The integration between Coveralls and Travis-CI is extremely straight forward since [goveralls](https://github.com/mattn/goveralls) is available. The usage of goveralls is well documented at [README](https://github.com/mattn/goveralls/blob/master/README.md).

I think using services are a great way to get started and also keep costs down. I'm glad that I have added Coveralls since it is a great way to get feedback on how your projects are trending.
