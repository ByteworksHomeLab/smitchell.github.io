---
layout: sidenote
title:  "How to Boot from USB"
url: /boot-from-usb
comments: true
date: 2020-11-29
categories: side-notes
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 15
show_related_posts: false
feature_image: feature-sidenotes
square_related: recommend-sidenotes
---
In the fall of 2020, Raspberry Pi released the long-awaited Raspberry Pi 4 boot EEPROM to boot directly from a USB3 drive without having the MicroSD card plugged in. [Jeff Geerling](https://www.jeffgeerling.com/blog) has a blog and video showing the upgrade: [I'm booting my Raspberry Pi 4 from a USB SSD](https://www.jeffgeerling.com/blog/2020/im-booting-my-raspberry-pi-4-usb-ssd).

# Prepare the Raspbian microSD
If you are updating multiple Raspberry Pis, like me, these steps only have to be done once to prepare the microSD card. Reuse the microSD on any Raspberry Pis that need the EEPROM update. Only the last two steps, “rpi-eeprom-update” and “vcgencmd,” get repeated after updating the first Raspberry Pi.

Flash a MicroSD card with Raspbian. I use Raspbian lite, the server version. Be sure to add the ssh file after flashing.
```shell
touch /Volumes/boot/ssh
```

Eject the microSD, insert it into the Raspberry Pi, power it up, and wait a bit for it to come online. Use ping to find the IP address on your network.

```shell
ping -c 2 raspberrypi
```

Connect to the IP address using ssh, and log in as user “pi” using the password “raspberry.”

# Updating the EEPROM

If you have an older Raspberry Pi 4, you need to upgrade the boot loader EEPROM to enable booting from USB drives. Update Rasbian to get the latest boot loader, and then edit “/etc/default/rpi-eeprom-update” to allow the Pi to use the latest stable version.

```shell
sudo apt update
sudo apt full-upgrade -y
sudo vi /etc/default/rpi-eeprom-update
```

Change the status from “critical” to “stable.”

{% include image.html url="/img/post-assets/sidenote-boot-from-usb/firmware_release_status.png" description="File rpi-eeprom-update" %}

Now, find the latest EEPROM version. 

```shell
ls /lib/firmware/raspberrypi/bootloader/stable/pieeprom-*
```

{% include image.html url="/img/post-assets/sidenote-boot-from-usb/eprom_versions.png" description="Stable EEPROM Versions" %}

Install the latest one; in my case, that is 2020-09-03.
```shell
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/stable/pieeprom-2020-09-03.bin
```
{% include image.html url="/img/post-assets/sidenote-boot-from-usb/updating_eprom.png" description="Updating the EEPROM" %}

Reboot, reconnect, then verify that the bootloader is up to date.
```shell
vcgencmd bootloader_version
```
{% include image.html url="/img/post-assets/sidenote-boot-from-usb/verify_bootloader.png" description="Verify the Boot Loader Update" %}

Use the same microSD card to update any other Raspberry Pis that need the new EEPROM. You only have to repeat the "rpi-eeprom-update" and "vcgencmd" steps for additional Raspberry Pi. Luckily, changing the boot firmware was a one-time change. 

 ----
 # References
 * * [I'm booting my Raspberry Pi 4 from a USB SSD](https://www.jeffgeerling.com/blog/2020/im-booting-my-raspberry-pi-4-usb-ssd)
