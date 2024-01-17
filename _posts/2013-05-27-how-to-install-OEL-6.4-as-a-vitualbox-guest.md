---
layout: post
title:  "How to Install OEL 6.4 as a VitualBox Guest"
url: /how-to-install-OEL-6.4-as-a-vitualbox-guest
comments: true
date: 2013-05-27 20:09:00
categories: geospatial
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
feature_image: feature-geospatial
show_related_posts: false
square_related: recommend-geospatial
youtubeid: lW3zlj3zWjM
---
Do you have much study time available at your job? Too
often, general research time is a luxury unless required for your current project. The best chance to expand my geospatial knowledge beyond what I already use at work is at home.

This weekend, I set out to build a test platform at home using Oracle 11g. I already had a development machine running Linux Mint, but I decided to use OEL 6.4 ([Oracle Enterprise Linux](http://www.oracle.com/us/technologies/linux/product/specifications/index.html)) instead to reduce the chance of OS incompatibilities with 11g. I re-imaged my computer with OEL but soon discovered that OEL is a better server OS than it is a desktop OS. My favorite web browser, Chrome, [doesn't currently work on OEL 6.4](http://www.h-online.com/open/news/item/Chrome-stops-declaring-Linux-systems-obsolete-1803451.html).

That made me switch to Ubuntu 12.04 as my base OS, with Oracle Enterprise Linux as a guest under Oracle VirtualBox. Virtualization has the advantage of supporting multiple database versions, which will come in handy when Oracle 12c is released. It also allows me to support other images, like PostGIS running under Centos 6.4, as pictured below.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/spatiallab1.png" description="Geospatial Home Lab" %}

I created custom partitions for the Ubuntu installation, in part because my machine has two drives, a 160 GB boot drive and a 500 GB storage drive, and in part to be able to preserve and reuse my `/home` and `/usr/local` partitions from one Linux installation to the next.

I created the swap, `/usr/local,` and root partition on the boot drive.

| MOUNT      | Partition Size |
|------------|----------------|
| swap       | 8 GB           |
| /usr/local | 5 GB           |
| /          | 147 GB         |

I allocated the entire second drive to '/home' because that is where VirtualBox stores virtual storage files.

| MOUNT | Partition Size |
|-------|----------------|
| /home | 500 GB         |

VirtualBox is a free download from the Ubuntu Software Center, as pictured here.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/virtualbox.png" description="VirtualBox" %}

Launch VirtualBox and click New. VirtualBox was smart enough to default the OS to Linux and the version to Oracle when I typed "OEL" in the name.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm2.png" description="VM Name and OS Type" %}

Assign memory to the virtual machine. The [minimum memory is 1 GB, but 2 GB](http://docs.oracle.com/cd/E11882_01/install.112/e24321/pre_install.htm#CHDCEBFF) is recommended. OEL 6.4 [requires a minimum of 1 GB of memory](http://www.oracle.com/us/technologies/linux/product/specifications/index.html). I gave the OEL Guest 2 GB of memory.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm3.png" description="Memory" %}

Create a new virtual hard disk file.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm4.png" description="Storage" %}

Accept the default file type.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm5.png" description="Virtual Disk Creation Wizard" %}

Accept the default virtual storage option to grow the virtual disk file as needed dynamically.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm6.png" description="Virtual Disk Creation Wizard" %}

Oracle Database EE [requires about 5 GB of storage](http://docs.oracle.com/cd/E11882_01/install.112/e24321/pre_install.htm#CHDIFGAI). Since this database is just for learning purposes, it will never house much data. I decided to give the OEL 6.4 guest image a maximum of 40 GB, allocated only as needed. That gives me plenty of physical disk space for multiple image snapshots, database versions, and other operating system images.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/40gb.png" description="File and Location" %}

VirtualBox allows you to review your settings before creating the guest image.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm8.png" description="Summary" %}

After creating the OEL guest, I opened the Network setting and changed the "Attached to" setting to Bridged Adapter. The bridged adapter allowed me to configure my firewall's DHCP server to assign a static IP address to OEL using its MAC address. At this point, I also edited the MAC address assigned to the guest to match one setup on my firewall during a previous install this weekend.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vm9.png" description="Bridged Network Connection" %}

Here is an example of the setting on my firewall's DHCP server to assign static IP 192.168.1.39 to MAC address 08:00:27:A8:CA:DC. The OEL NIC is set up as DHCP but always gets the same IP address.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/staticip.png" description="Static IP Settings" %}

All that remains is to mount the OEL 6.4 ISO images. The smaller image is the boot disk. The 3.5 GB image contains the installation source. Attach both ISO images to the IDE Controller. Ignore the black box that says "SATA Controller." I didn't mean to capture that in the screenshot.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/screenshot-from-2013-05-30-145643.png" description="Virtual Machine Storage" %}

## Installing OEL 4.6

Start the VirtualBox guest, and the following screen will appear. Pick the first option.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel1.png" description="Install OEL" %}

Press TAB ENTER to Skip the media test.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel2.png" description="Find Media" %}

The installer will detect the second-mounted ISO image
and the installation will continue.

On the next couple of screens, not shown, choose a language for the installation process and choose a keyboard type for the system.

Select the device type for the installation.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel42.png" description="Storage Devices" %}

There are two crucial settings on the next page. Assign a hostname for the virtual machine, and then click the Configure Network button to edit eth0. You'll want to select Connect automatically.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/hostname.png" description="Hostname" %}

Pick a time zone for your server.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel7.png" description="Timezone" %}

Assign a password for root.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel8.png" description="Password" %}

If you are using a newly created virtual storage file, selecting the first option to use all space is safe.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel10.png" description="Disk Space" %}

Pick the installation type. I chose a desktop just because I thought it would be easier for a learning machine.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/defaultinstallation.png" description="Default Installation" %}

Okay, it's time for coffee while all the packages get installed.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/defaultinstallation.png" description="Installing..." %}

I just happened to capture the message above about setting the SELinux policy. We'll change that when we get to the database setup.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel12.png" description="Installing Complete" %}

You successfully installed an OEL 6.4 guest on Oracle VirtualBox.

## Install Guest Additions

If you want the convenience of a shared clipboard, shared folders, and quality display settings, you are not done yet. To get those things, you need to install VBox Additions. Start the OEL 6.4 guest and hover over the guest window's toolbar to get the VirtualBox menu to appear at the top of the Ubuntu desktop. Move the mouse over the VirtualBox Devices menu and then go to the Install Guest Additions sub-menu. Follow the prompts, and VirtualBox will mount a VBOX Additions ISO image on the OEL desktop.

Sign into your guest and switch to the root user. Before you can run the installation, you need to install the following packages in OEL 6.4.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/oel12.png" description="Installing Complete" %}

I just happened to capture the message above about setting the SELinux policy. We'll change that when we get to the database setup.

```shell
yum update
yum install kernel* dkms gcc
reboot
```
You can double-click on the VirtualBox Additions CD-ROM image on the OEL desktop.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/autorun.png" description="Autorun VirtualBox Additions" %}

You should see console output similar to this.

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/vboxadd3.png" description="VirtualBox Guest Additions Installation" %}

You can verify the Virtualbox Guest Additions by copying something to the clipboard in Ubuntu and pasting it in OEL. Likewise, if you option the VirtualBox guest settings and define a shared folder with the auto-mount option, then in OEL, you should find the shared folder mounted under the /shared/media directory.

In my next post, we'll install Oracle Database Enterprise Edition on our OEL 6.4 guest.




