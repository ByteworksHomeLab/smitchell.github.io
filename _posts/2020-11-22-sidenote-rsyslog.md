---
layout: sidenote
title:  "How to Centralize Logs with rsyslog"
url: /how-to-centralize-logs-with-rsyslog
comments: true
date: 2020-11-22
categories: side-notes
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
show_related_posts: false
feature_image: feature-sidenotes
square_related: recommend-sidenotes
---
Whoops, I did it again! I lost access to my primary node several times, but I never knew why because the logs weren’t accessible. I need centralize logging outside of the cluster. 

{% include image.html url="/img/post-assets/sidenote-rsyslog/rsyslog_flow.png" description="Centralized Logs with rsyslog" %}

# Cloud Logging
My first instinct was to use cloud logging, but that proved too costly or too complicated.

## SolarWinds Loggly

Loggly works perfectly on Raspberry Pis. It has a 30-day trial subscription. I chose not to continue because the least expensive month-to-month subscription is $99/month.

{% include image.html url="/img/post-assets/sidenote-rsyslog/loggly.png" description="SolarWinds Loggly Dashboard" %}

## AWS CloudWatch Logging
AWS CloudWatch is easy to set up on the server-side. There is a compatible agent, too, collectd, that sends metrics to CloudWatch. The collectd agent is from AWSLabs and is written in Python, making it portable to the Raspberry Pi. 

Unfortunately, the only CloudWatch Logging Agent ARM package for Debian is arm64. Raspbian is armhf, so the agent doesn’t work. I did not find a way to stream logs to AWS CloudWatch from Raspbian.

## Google Cloud Platform Logging
You can send on-premise logs to GCP using Fluentd, and it is possible to run Fuentd on a Raspberry Pi, but I was unable to do so with GCP. This article documents sending logs from Raspbian to Treasure Data, a very pricy service:  <a href="https://docs.fluentd.org/v/0.12/articles/raspberrypi-cloud-data-logger">Raspberry Pi Cloud Data Logging</a>. Honestly, when I sat down to work on this post, I couldn’t remember what didn’t work with GCP logging, but trust me, I tried.

# Using Rsyslog to Centralize Logs
In the end, I decided to centralize my logs on a spare MacMini running Ubuntu. Rsyslog is very easy to setup. I don’t have Kabana or even a UI, but separate log files are centrally available should I need them to troubleshoot another lost node.

Here is the blog that I followed to configure Rsyslog: <a href="https://computingforgeeks.com/how-to-configure-rsyslog-centralized-log-server-on-ubuntu-18-04-lts/">How to Configure Rsyslog Centralized Log Server on Ubuntu 18.04 LTS</a>

## Rsyslog Server
   
First, you uncomment the udp and tcp port binding lines in the /etc/rsyslog.conf file. Next, you add a template for writing the remote incoming logs. See the post referenced above for all the details.

{% include image.html url="/img/post-assets/sidenote-rsyslog/rsyslog.png" description="Rsyslog Server Configuration" %}

## Rsyslog Client Setup
   
Add the configuration for the remote Rsyslog server to the /etc/rsyslog.conf file on all the Raspberry Pi nodes. Again, see the post referenced above for the details.

{% include image.html url="/img/post-assets/sidenote-rsyslog/rsyslog_client.png" description="Rsyslog Client Configuration" %}

Also, the syslog is excluded by default. I'm sure a Linux admin knows a good reason for that, but for my home lab I need the logs. In order to send syslog you need to remove the dash in front of it.

{% include image.html url="/img/post-assets/sidenote-rsyslog/syslog.png" description="Include syslog" %}

Finally, the linked instructions I followed are for Ubuntu, so it has the wrong command to restart Rsyslog on Raspbian. Use this command instead.

```shell 
sudo service rsyslog restart
```

After restarting Rsyslog on all the Raspberry Pis, you should see directories matching the Pi hostnames appear on the Rsyslog server in the /var/log directory, as shown below:

{% include image.html url="/img/post-assets/sidenote-rsyslog/centralized_logs.png" description="Rsyslog Server /var/log Directories" %}

## Defining Log Rotation
Add log rotation rules to avoid filling up the hard disk. This is done with configuration files in the /etc/logrotate.d/ directory.

{% include image.html url="/img/post-assets/sidenote-rsyslog/log_rotate.png" description="Log Rotation Configuration Files" %}

Each configuration file looks something like this.

```shell
/var/log/pi1/*.log {
        weekly
        missingok
        dateext
        rotate 3
        size=10M
        create 0644 root root
        notifempty
        sharedscripts
}
```
That’s all there is to it. When I figure out how to get cloud logging working with Azure, GCP, or AWS, I will add a new logging post.

----
## References
* <a href="https://itnext.io/creating-a-full-monitoring-solution-for-arm-kubernetes-cluster-53b3671186cb">Creating a full monitoring solution for ARM Kubernetes Cluster</a>
* <a href="https://linoxide.com/linux-how-to/setup-log-rotation-logrotate-ubuntu/">How to Setup Log Rotation with Logrotate on Ubuntu 18.04</a>
* <a href="https://computingforgeeks.com/how-to-configure-rsyslog-centralized-log-server-on-ubuntu-18-04-lts/">How to Configure Rsyslog Centralized Log Server on Ubuntu 18.04 LTS</a>

