---
layout: post
title:  "Project LCA"
date:   2016-08-16 20:15:00 +1200
---
If you follow my GitHub, you may have noticed I have super simple REST API written in Go called [level-three-rest](https://github.com/bhavikkumar/level-three-rest). This repository is going to form the basic building block of my project.

There is still a lot of work which needs to be done on the level-three-rest project before I can start building out the rest of this project.

## The Goal
The eventually goal of this project is to have a application which resembles something that an actual business would run in production.

I think it will be a fantastic way to learn and can be a realistic demo one day.

## What will the project consist of?
At this stage I am thinking of the following at a high level.
 - Multiple microservices
  - With some dependencies between them.
 - A Front-End
  - With authentication and authorisation
 - Containers
 - Service Discovery
 - Configuration Discovery

In the end if I can have these additional items I think it will be great:
- Auto Scaling
- Analytics
- Chaos Monkey
- Automated Integration tests

I am also attempting to keep as much of the microservices RFC compliant. I think this is a project which will definitely be a challenge but something I can also achieve.

## Why not a serverless project?
You mean serviceful right? I'm going to call it serviceful. The reason is a simple one, the tooling is still evolving and I don't think there are many existing businesses which are ready to adopt a completely serviceful architecture.

Maybe the next project can be migrating this project to a serviceful architecture.
