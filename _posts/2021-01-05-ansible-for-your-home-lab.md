---
layout: post
title:  "Ansible for your Home Lab"
url: /ansible-for-your-home-lab
comments: true
date: 2021-01-02 14:17:00
categories: raspberry python
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 10
feature_image: feature-raspberry
show_related_posts: false
square_related: recommend-raspberry
---
{% include image.html url="/img/post-assets/ansible-for-your-home-lab/ansible.png" %}

Disaster struck in my home lab, again. This time I knocked a power cord loose on my USB power hub, taking down four of my Kubernetes nodes. That usually would not have been a “disaster,” but it broke the “sudo” command, and you can’t maintain Ubuntu without the sudo command. The sudo command complained that it couldn't resolve the hostname because I didn’t add the hostname to the /etc/hosts file when I changed the hostname earlier.

There are nine Raspberry Pis in my home lab that first ran Raspbian, then Ubuntu, and I rebuilt for other reasons along the way. I decided now was a good time to add some automation.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/stack.png" description="Home Lab Raspberry Pis" %}

# Ansible
Michael DeHaan released Ansible in 2012, and RedHat acquired it in 2015. It is an open-source tool written in python, power shell, and ruby that provisions infrastructure, networks, containers, security, cloud, and more. Ansible is one of the ways companies create infrastructure as code.

It is easy to get started with Ansible because there is nothing to install on the target systems. Simply install it on your computer and start automating. Redhat sells Ansible Tower for the enterprise, but it is not required to use Ansible.

I first used Ansible in 2016 when the startup where I worked rehosted our DC/OS (Data Center Operating System) cluster in the cloud. Our three-person team did a cloud bake-off to decide which cloud to use. I took GCP (Google Cloud Platform), which happened to have a DC/OS Ansible playbook. It made deploying DC/OS to GCP so easy, and therefore we decided to use it.

{% include tip.html content="Learn how we deployed DC/OS to GCP using Ansible: <a href='/dcos-on-gcp'>Jump to the article</a>." %}

Check out [How Ansible Works](https://www.ansible.com/overview/how-ansible-works) to learn more.

# Home Lab Ansible Playbook

Despite using Ansible off and on since 2016, I never wrote playbooks until now. I set out to replace some of the manual setup I perform. The steps to set-up Ubuntu on Raspberry Pi are covered in my previous post [Running Ubuntu on a Raspberry Pi Cluster](/running-ubuntu-on-rpi). Here they are:

1. Flash the drive with the Ubuntu server image.
1. Add the ssh file.
1. Change the password at the first login.
1. Add the ssh key on the host as an authorized key.
1. Update and upgrade the Ubuntu packages.
1. Change the hostname.
1. Add the hostname to the /etc/hosts file.
1. Set the static IP address.

My Ansible playbook only handles the last four steps. I ran into a problem with the first login’s mandatory password change. Some Ansible playbooks use the sshpass command to change the password, but I could not install sshpass on my Mac using Brew. Long story.

## Inventory
First, to set up, Ansible needs an inventory of the new hosts. My playbook requires a couple of variables for each host. Using my network scanner, I found the DHCP addresses of the new host as it booted and added the host IP address to the list. Next, I performed the first four setup steps manually, just as before. 

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/inventory.png" description="Ansible Hosts Inventory with Variables" %}

Each host in the “new_host” group gets a “new_ip” and “new_host” variable, needed by my Ansible playbook. In the image above, you see that the static IP address, 192.168.1.50, is given to the first host, and its hostname is pi5.

## Playbook
My playbook includes seven tasks.

1. Update Ubuntu.
1. Change the hostname.
1. Update the /etc/hosts file.
1. Disable Netplan automatic configuration.
1. Update the Netplan configuration with the static IP address.
1. Apply the static IP Netplan configuration.
1. Reboot the host.

### 1) Update Ubuntu
First, make sure that Ubuntu is up-to-date.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/apt_upgrade.png" description="Apt Update and Upgrade" %}

### 2) Change the Hostname
Change the hostname using the “new_host” variable specified in the inventory file.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/set_host_name.png" description="Change the Hostname" %}

### 3) Replace the /etc/hosts file
Add the hostname specified in the “new_host” variable to the /etc/hosts file. The playbook uses Jinga2 to replace a variable in the template file created from the default /etc/hosts file.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/copy_hosts_file.png" description="Template Task to Replace /etc/hosts" %}

### 4) Disable Automatic Netplan Configuration
This task places a file in the /etc/netplan directory to disable the automatic generation of the Netplan configuration.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/disable_network_config.png" description="Disable Automatic Netplan Configuration Generation" %}

### 5) Add the Static IP Address to the Netplan Configuration
This task uses Jinja2 to replace the host IP in a Netplan configuration template with the “new_ip” variable from the Ansible host inventory.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/setup_static_ip.png" description="Replace the Netplan Configuration using the Static IP Template" %}

### 6) Apply the Netplan Configuration
The purpose of this task is to apply the network configuration. I should also add a step to generate and debug the format, but I didn’t find an example.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/apply_netplan.png" description="Apply the Netplan Configuration" %}

### 7) Reboot the Host
Finally, the Ansible playbook reboots the host.

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/Reboot.png" description="Reboot the Host" %}

Ansible didn’t save me time today since I was learning Ansible as I went, but now I have an automated playbook to save me time and avoid future mistakes.
Here is the whole playbook:

{% include image.html url="/img/post-assets/ansible-for-your-home-lab/playbook.png" description="Ansible Playbook" %}

----
# References
* [Ansible Documentation](https://docs.ansible.com/?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW)
* [How Ansible Works](https://www.ansible.com/overview/how-ansible-works)
* [Running Ubuntu on a Raspberry Pi Cluster](https://smitchell.github.io/running-ubuntu-on-rpi)
