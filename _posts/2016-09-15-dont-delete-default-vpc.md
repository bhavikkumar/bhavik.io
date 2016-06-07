---
layout: post
title:  "Don't delete the default VPC in AWS"
date:   2016-06-15 05:00:00 +1200
---
I did a really silly thing a while ago, which was to delete my default VPC in AWS. This didn't affect me until recently as I have custom VPC's which I was launching EC2 instances into. The problem arose when I tried to run the beta of cloud former which tries to by default spin up a EC2 instance in the default VPC.

I don't really have a need for a developer support subscription, but I found out that this is a common problem and posting on the AWS forums, they will happily recreate a default VPC for you.

I've learnt my lesson though, never delete the default of anything!