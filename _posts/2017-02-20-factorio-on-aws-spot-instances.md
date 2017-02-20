---
layout: post
title:  "Factorio server on AWS spot instances"
date:   2017-02-22 16:00:00 +1200
published: false
---
Spot instances suit certain workloads such as analytics and image processing, however I thought it would be a good idea to see if spot instances cane be used as a gaming instance. The game being Factorio which can be played multiplayer and the server saves / pauses the game when there is no one playing on it. At first we were just playing with one of us hosting it on our machines, but to continue from where we left off someone would have to host the server and we would all have to be playing at the same time.

The original solution was to spin up a t2.nano and get the headless version running on it, this worked fine until one day when we noticed the server would keep crashing once the third person joined. It turns out the issue was the instance did not have enough memory to handle the growing size of the map. The cost of going to a t2.medium is about ~US$45 in Sydney region, at the time of writing this. That is pretty expensive for a casual gaming server.

The answer is to spin up a spot instance in a way where if it was terminated for any reason then another spot instance would automatically spin up and allow us to continue from the last auto save point. Running an EC2 instance this way meant we would be able to get a m3.medium instance for about ~80% less than a t2.medium on demand instance.

## The Options
There are a few options which we can use as long it achieves the following:

 - The game must automatically start when the instance is ready.
 - There must be a way to update the game to the latest version.
 - Auto saves must persist across instance termination

### Option 1 - EFS
The first option we considered is running spot instances which mount an AWS EFS volume to it which contain the game and saved games. This would of been the preferred option, the only problem was that EFS is not available in the Sydney region at the time of writing this, therefore latency would of been too high for certain elements of the game, such as driving the car or tank. Hopefully, EFS comes to the Sydney region at some point.

### Option 2 - EBS
The second option is to have a EBS volume with the game data already on it, this means spinning up a EC2 instance and then mounting another volume to it, copying the appropriate files on to it and configuring the server settings. The next steps is to detach the volume and terminate the instance which was used to setup the volume. After this I used the this CloudFormation script to setup the gaming server which auto mounts the user data and then runs a simple shell script which starts the game and also ensures that the game restarts if it crashes for some reason.

The only downside about this method is having to manually update the game when a new version comes out, luckily that is not too often.

#### Game start up script
{% highlight bash %}
#!/bin/sh
ps auxw | grep "factorio --start-server-load-latest" | grep -v grep

if [ $? != 0 ]
then
  while true; do
    echo "Starting factorio..."
    sudo su - factorio -c "/factorio/factorio/bin/x64/factorio --start-server-load-latest --server-settings /factorio/factorio/data/baa-server-settings.json"
    echo "Factorio crashed."
  done
fi
{% endhighlight %}

#### User Data
{% highlight shell %}
#!/bin/bash
# Start NTPD
service ntpd start

# Get the EC2 credentials
instance_profile=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/`
aws_access_key_id=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${instance_profile} | grep AccessKeyId | cut -d':' -f2 | sed 's/[^0-9A-Z]*//g'`
aws_secret_access_key=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${instance_profile} | grep SecretAccessKey | cut -d':' -f2 | sed 's/[^0-9A-Za-z/+=]*//g'`
aws_session_token=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${instance_profile} | grep Token | cut -d':' -f2 | sed 's/[^0-9A-Za-z/+=]*//g'`

export AWS_ACCESS_KEY_ID=${aws_access_key_id}
export AWS_SECRET_ACCESS_KEY=${aws_secret_access_key}
export AWS_SESSION_TOKEN=${aws_session_token}
export AWS_DEFAULT_REGION=<region>

# Get the instance Id
export EC2_INSTANCE_ID=`curl http://169.254.169.254/latest/meta-data/instance-id`

# Attach the volume
aws ec2 attach-volume --volume-id <vol-id> --instance-id $EC2_INSTANCE_ID --device /dev/xvdf

# Wait until the volume is attached before mounting the volume
DATA_STATE="unknown"
until [ $DATA_STATE == "attached" ]; do
  DATA_STATE=`aws ec2 describe-volumes --filters Name=attachment.instance-id,Values=$EC2_INSTANCE_ID Name=attachment.device,Values=/dev/xvdf --query Volumes[].Attachments[].State --output text`
  sleep 5
done

# Mount the volume
mkdir /factorio
mount /dev/xvdf /factorio

# Create Factorio User and start the server
useradd -b /factorio factorio
/factorio/start-factorio.sh > /dev/null &
{% endhighlight %}
