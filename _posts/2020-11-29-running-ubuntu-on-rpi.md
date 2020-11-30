---
layout: post
title:  "Running Ubuntu on a Raspberry Pi Cluster"
url: /ubuntu-on-rpi
comments: true
date: 2020-11-29 08:42:00
categories: kubernetes raspberry
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
show_related_posts: false
feature_image: feature-raspberry
square_related: recommend-raspberry
ubuntu_launch: 0pT4-RcTERU
ubuntu_static: gJtZOlYR_MM
---
When I started using Kubernetes on 32-bit Raspbian I quickly discovered that many Docker images are not compatible. To avoid spending time rebuilding Docker images for linux/amd, I decided to switch to an AMD64 OS. There is an [experimental AMD64 version of Raspbian in Beta](https://www.raspberrypi.org/forums/viewtopic.php?t=275370), but I chose my old friend, Ubuntu.

{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/ubuntu+rpi.png" description="Ubuntu on Raspberry Pi" %}

On October 22, 2020, Canonical released an [Ubuntu Desktop image optimised for the Raspberry Pi](http://ubuntu.com/raspberry-pi). What most excited me about this release is support for the AMD64 platform on the Raspberry Pi 3 and 4.

{% include youtubePlayer.html id=page.ubuntu_launch %}
Ubuntu Desktop for Raspberry Pi

Before I install Ubuntu, I am updating the boot loader to boot directly off of USB without needing the microSD card installed.

# Booting from a USB Drive

In the fall of 2020, Raspberry Pi officially released the long awaited Raspberry Pi 4 boot EEPROM to boot directly from a USB3 drive without having the MicroSD card plugged in. [Jeff Geerling](https://www.jeffgeerling.com/blog) has a blog and video showing the upgrade: [I'm booting my Raspberry Pi 4 from a USB SSD](https://www.jeffgeerling.com/blog/2020/im-booting-my-raspberry-pi-4-usb-ssd).

## Prepare the Raspbian microSD
If you are updating multiple Raspberry Pis, like me, these steps only have to be done once to prepare the microSD card. Reuse the microSD on any Raspberry Pis that need the EEPROM update. Only the last two steps, “rpi-eeprom-update” and “vcgencmd,” get repeated after updating the first Raspberry Pi.

You need a MicroSD flashed with Raspbian. I use Raspbian lite, the server version. Be sure to add the ssh file.
```shell
touch /Volumes/system-boot/ssh
```

Eject the microSD, install it in the Raspberry Pi, power it up, and wait a bit for it to come online. Use ping to find the IP address.

```shell
ping -c 2 raspberrypi
```

Connect to the IP address using ssh, and log in using the password “raspberry.”

## Updating the EEPROM

If you have an older Raspberry Pi 4, then you need to upgrade the boot loader EEPROM to enable booting from USB drives. These are the steps I followed after my first login. 

```shell
sudo apt update
sudo apt full-upgrade -y
sudo vi /etc/default/rpi-eeprom-update
```

Change the status from “critical” to “stable.”

{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/firmware release_status.png" description="File rpi-eeprom-update" %}

Now, find the latest EEPROM version. 

```shell
ls /lib/firmware/raspberrypi/bootloader/stable/pieeprom-*
```

{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/eprom_versions.png" description="Stable EEPROM Versions" %}

Install the latest one; in my case, that is 2020-09-03.
```shell
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/stable/pieeprom-2020-09-03.bin
```
{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/updating_eprom.png" description="Updating the EEPROM" %}

Reboot, reconnect, then verify that the bootloader is up to date.
```shell
vcgencmd bootloader_version
```
{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/verify_bootloader.png" description="Verify the Boot Loader Update" %}

You are now ready to boot from your USB drive.
1. Shutdown then power off the Raspberry Pi.
1. Remove the microSD card.
1. Attach a USB3 drive flashed with whatever OS you chose.
1. Turn the power back on.

Use the same microSD card to boot any other Raspberry Pis that need an EEPROM update. You only have to repeat the "rpi-eeprom-update" and "vcgencmd" steps for each upgrade. Luckily, changing the boot firmware was a one-time change. 

Now that the boot firmware is up-to-date, let’s install Ubuntu. 

# Setting up Ubuntu

Getting the host ready for the cluster is easy now that it boots directly from the USB drive. All we have to do the following:
1. Flash a USB drive with Ubuntu.
1. Install the latest updates.
1. Set the static IP address
1. Change the hostname.
1. Add SSH keys for authentication.

## Flashing the USB Drive

Plug a USB drive into your desktop or laptop. Open the Raspberry Pi Imager and select the distribution you desire. I run my servers headless, so I chose Ubuntu Server 20.10 (RPI 3, 4, 400) for AMD64. Select your USB drive under “SD Card,” and click Write.

{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/ubuntu_flash.png" description="Flash the USB Drive with Ubuntu" %}

Remount the USB drive after it is flashed and add the ssh file.

```shell
touch /Volumes/system-boot/ssh
```

## Boot Ubuntu
Give the system a minute to reboot then find the IP address on your network and connect via ssh. Use “ping -c 2 ubuntu” to find the IP.

{% include warning.html content="I could not immediately connect with SSH to one of my eight Raspberry Pis, even though I added the ssh file after it was flashed. If this happens to you, attach a monitor and keyboard, login, and reset the password. SSH should work after you reboot." %}


## Update Ubuntu

Update the Ubuntu OS after your first login.

```shell
sudo apt update
sudo apt full-upgrade -y
```

## Setting the Static IP

Here is a six-minute video showing how to set a static IP using Netplan.

{% include youtubePlayer.html id=page.ubuntu_static %}
Setting Static IPs with Netplan

Before setting the static IP, disabled cloud-init’s network configuration capabilities by adding this file: 
/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg. Include the line below in the file.

```shell
network: {config: disabled}
```

Make a backup copy of the file “/etc/netplan/50-cloud-init.yaml,” then add the static IP information. Here is how I configured my 50-cloud-init.yaml file.

```shell
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
  version: 2
  ethernets:
    eth0:
        dhcp4: false
        addresses: [192.168.1.10/24]
        gateway4: 192.168.1.1
        nameservers:
          addresses: [192.168.1.1, 8.8.8.8]
        match:
          :wdriver: bcmgenet smsc95xx lan78xx
        optional: true
        set-name: eth0
```

Validate the YAML file.
```shell
sudo netplan --debug generate
```
{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/netplan_debug.png" description="Verify the Netplan" %}

If the file is valid, apply the changes and reboot; however, if your keyboard is still attached, use it instead of doing this with an ssh connection since the IP address will change. 

```shell
sudo netplan apply
```

Reboot and connect to the host on its new IP address.

## Changing the Host Name

Give the host a unique name. My cluster hostnames are pi1 - pi8. Use the hostnamectl command to make the change.

```shell
sudo hostnamectl set-hostname pi1
```
{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/hostname.png" description="Change the Hostname" %}

## Adding SSH Key for Authentication

Add your computer’s SSH key to the host to not have to enter a password when you connect with SSH.

{% include tip.html content="This tip shows how to set up SSH and multicast commands to all the nodes in the cluster: <a href='/how-to-multicast-commands'>How to Multicast Commands</a>. " %}

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub  ubuntu@pi1
```

That’s it. We’re now ready to install Rancher K3s Kubernetes. Here is the result

{% include image.html url="/img/post-assets/2020-11-29-running-ubuntu-on-rpi/UbuntuAMD64Pis.png" description="Ubuntu on Pis" %}

----
# References
* [I'm booting my Raspberry Pi 4 from a USB SSD](https://www.jeffgeerling.com/blog/2020/im-booting-my-raspberry-pi-4-usb-ssd)
* https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview
* https://www.youtube.com/watch?v=gJtZOlYR_MM
* https://www.youtube.com/watch?v=0pT4-RcTERU
* [Build a Raspberry Pi Desktop with an Ubuntu heart](https://ubuntu.com/blog/build-a-raspberry-pi-desktop-with-an-ubuntu-heart)









