---
layout: post
title:  "Running Kubernetes on a Raspberry Pi Cluster"
url: /k8s-on-rpi4
comments: true
date: 2020-11-22 14:42:00
categories: kubernetes raspberry
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 10
show_related_posts: false
feature_image: feature-raspberry
square_related: recommend-raspberry
---
I loved trains as a boy, which may explain why I am running Kubernetes on Raspberry Pis for my home lab instead of in the cloud. Think of it like model railroading for computer geeks! In my earlier post, <a href="https://smitchell.github.io/how-to-configure-raspbian-linux">How to Configure Raspbian Linux</a>, we installed and configured Raspbian Linux on all Raspberry Pis. Now it’s time to add Kubernetes.

{% include image.html url="/img/post-assets/2020-11-22-k8s-on-rpi/locomotives.png" description="Kubernetes and Raspbian Locomotives" %}

{% include tip.html content="<a href='https://www.alexellis.io/'>Alex Ellis</a> has an excellent post about installing Rancher Labs K3s on your Raspberry Pi cluster called <a href='https://blog.alexellis.io/test-drive-k3s-on-raspberry-pi/'>Will it cluster? k3s on your Raspberry Pi</a>." %}

# Choosing a Kubernetes Distribution
You have choices when it comes to a Kubernetes distribution for your Raspberry Pis. Here are just a few.

* Canonical Micro8s
* Minimal Kubernetes via kubeadm
* Rancher Labs K3s

{% include image.html url="/img/post-assets/2020-11-22-k8s-on-rpi/k3s.png" description="Rancher Labs k3s" %}

I selected Rancher for its popularity in the Raspberry Pi community. K3s is a highly available, certified Kubernetes distribution, yet the k3s binary is less than 40MB. The sweet spot for k3s is IoT (Internet of Things) and Edge devices. For instance, many national retailers put k3s in every store to run point-of-sale and other operational systems. Most importantly for our Raspberry Pi cluster is that K3S works great on ARM processors!

One of the differences in k3s is that Rancher replaced the etcd Kubernetes database with SQLite, allowing packaging of k3s as a single binary. It also allows you to run in HA (high availability) mode with a single master node using an external SQLite database cluster. K8s can’t do that. You can also achieve HA with multiple k3s master nodes, the same as k8s.

## Installing K3s

If you decide to install k3s on your own Raspberry Pi’s, follow these tips that walk you through the process.

{% include tip.html content="K3s installation instructions for the Raspberry Pi. <a href='/sidenote-k8s-on-raspbian'>Jump to article</a>" %}

## Installing a Kubernetes Dashboard
Kubernetes includes a dashboard, but you have options if you want something different.

{% include tip.html content="Learn about different Kubernetes dashboards and how to install them. <a href='/sidenote-dashboards'>Jump to article</a>" %}

## Centralized Logging
Things can go wrong with your Raspberry Pi nodes. Trust me; I have some first-hand experience; that is why I set up my cluster to send the logs off-node, enabling me to examine them separately if the node goes down.
   
{% include tip.html content="Centralized logging instructions for the Raspberry Pi. <a href='/sidenote-rsyslog'>Jump to article</a>" %}
   
That’s it for now. In my next post, I will begin deploying a workload to my new cluster.
   
Before I sign off, here is a peek at my current eight-node cluster as of Thanksgiving, 2020:

{% include image.html url="/img/post-assets/2020-11-22-k8s-on-rpi/cluster.png" description="Raspberry Pi Cluster" %}



