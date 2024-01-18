---
layout: post
title:  "Installing Oracle SQL Developer"
url: /installing-oracle-sql-developer
comments: true
date: 2013-06-08 21:48:00
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
<a href="./installing-oracle-sql-developer">
    <img 
        src="/img/post-assets/2013-06-08-installing-oracle-sql-developer/sqldevlogo.png" 
        alt="SQL Developer Logo"
    >
</a>

This short post covers the installation of Oracle SQL Developer and the IP tables change required on OEL 6.4 to open port 1521 so SQL Developer can connect to the Oracle database.

The illustration below from my earlier post, Host to Install OEL 6.4 as a VirtualBox Guest, shows that I run my Oracle 11g Enterprise Edition spatial learning lab on VirtualBox. Still, my tools run on the host OS, Ubuntu:

{% include image.html url="/img/post-assets/2013-05-27-how-to-install-OEL-6.4-as-a-vitualbox-guest/spatiallab1.png" description="Spaital Lab" %}

## Configure the Host File

Add the hostname of the Oracle database server to `/etc/hosts` on the machine running SQL Developer.

{% include image.html url="/img/post-assets/2013-06-08-installing-oracle-sql-developer/hosts.png" description="Hosts File" %}

## Open Port 1521

SQL Developer must connect to the TNS listener on the Oracle database server over port 1521 (or some other configured port), so Linux must accept traffic on port 1521. Use Telnet to test whether the port is open on the database server. SQL Developer issues the following command from the machine running, substituting your Oracle database server hostname and listener port.

```shell
$ telnet gps11g 1521
Trying 192.168.1.39...
telnet: Unable to connect to remote host: No route to host
```

If Telnet cannot get through on that port, check the /etc/sysconfig/iptables file on the Oracle database server. Confirm that port 1521 is open. I had to add the rule outlined in red. Please talk to your Systems Administrator about the risks of making this change. My house is behind a firewall appliance, and port 1521 is blocked from the outside world, so I felt secure making this change.

{% include image.html url="/img/post-assets/2013-06-08-installing-oracle-sql-developer/iptables.png" description="IPTables" %}

If you make changes to `/etc/sysconfig/iptables,` run the following command to restart iptables afterward:

```shell
$ /etc/init.d/
iptables restart 
iptables: Flushing firewall rules:                [ OK ] 
iptables: Setting chains to policy ACCEPT: filter [ OK ] 
iptables: Unloading modules:                      [ OK ] 
iptables: Applying firewall rules:                [ OK ]
```

## Download SQL Developer

Download SQL Developer from here: http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html. Pick the appropriate distribution for your system. For OEL 6.4, choose Other Platforms.

{% include image.html url="/img/post-assets/2013-06-08-installing-oracle-sql-developer/sqldev-download.png" description="SQL Download" %}

## Install SQL Developer

Read the complete installation documented here: https://forums.oracle.com/forums/thread.jspa?threadID=2302774. Follow the instructions found on that page to install SQL Developer.
The SQL Developer software requires Java. The link above includes a step to add the Java installation location to a configuration file. I've had to do that in the past on other machines, but I did not have to do that on this machine. Since I'm a Java developer and need to use an older Sun distribution of Java for compatibility reasons, I did a separate set-up step not mentioned before in my blog. That included adding JAVA_HOME to my .bashrc file.

## Add a Database Connection

After installing and launching SQL Developer, click the green plus sign to add a new database connection:

{% include image.html url="/img/post-assets/2013-06-08-installing-oracle-sql-developer/sqldevadd.png" description="Add Connection" %}

Supply the information displayed when the TNS Listener was started.

{% include image.html url="/img/post-assets/2013-06-08-installing-oracle-sql-developer/db_connection.png" description="DB Connection" %}

Connect to the Oracle Database Server
When your database starts, it listens on port 1521, and iptables allows traffic on port 1521. You should now be able to connect to the SPATIAL_LEARNING database.

{% include image.html url="/img/post-assets/2013-06-08-installing-oracle-sql-developer/db_data.png" description="Spatial Data" %}

In my next post, I will set up Oracle Map Viewer.
