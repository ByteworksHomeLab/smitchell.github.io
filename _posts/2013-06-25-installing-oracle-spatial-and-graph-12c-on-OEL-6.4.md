---
layout: post
title:  "Installing Oracle Spatial and Graph 12c on OEL 6.4"
url: /installing-oracle-spatial-and-graph-12c-on-OEL-6.4
comments: true
date: 2013-06-25 23:03:00
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
<a href="./installing-oracle-spatial-and-graph-12c-on-OEL-6.4">
    <img 
        src="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_logo.png" 
        alt="Oracle Spatial and Graph Logo"
    >
</a>

## Oracle Spatial and Graph 12c has Arrived.

Today, Jean Ihm posted on Google+ that [Oracle Spatial and Graph 12c is available](https://plus.google.com/u/0/108941302542585589628/posts/TAykn6tyYqX). This post is an update to my post earlier this month on [How to Install Oracle Spatial and Graph (11g) on OEL 6.4](http://exploringspatial.wordpress.com/2013/06/01/how-to-install-oracle-spatial-on-oel-6-4/). Bear in mind that the context of my blog is a home spatial learning lab written by a Java developer, not a DBA, so please take what I say with a grain of salt.

## Oracle 12c Gotcha

The Oracle 12c installation has "Create as Container database" checked by default. Container and PDB (Pluggable Database) didn't exist in 11g and are fundamental paradigm shifts. Unless you add that complexity to your installation, uncheck the box circled in red below on the fifth screen show.

## Complete the Pre-install Steps for Oracle Spatial and Graph 12c

I cloned an Oracle VirtualBox snapshot of OEL 6.4 (Oracle Enterprise Linux) just before installing Oracle 11g. I performed the prerequisite steps as documented in my 11g install post:
1. Define the hostname.
2. Install the dependencies. I didn't find a 12c pre-install RPM for OEL, so I used the 11g RPM: [oracle-rdbms-server-11gR2-preinstall](https://blogs.oracle.com/linux/entry/oracle_rdbms_server_11gr2_pre).
3. Change the secure Linux policy in /etc/selinux/config to permissive.

## Install Oracle Spatial and Graph 12c

Download the two Linux zip files from the [Oracle Spatial and Graph 12c](http://www.oracle.com/us/products/database/options/spatial/overview/index.html) page.

As root, create the directory shown below and unzip both files into that directory.

```shell
# mkdir /home/OraDB12c/
$ cp [your path]/linuxamd64_12c_database_1of2.zip /home/OraDB12c/ 
$ cp [your path]/linuxamd64_12c_database_2of2.zip /home/OraDB12c/ 
$ cd /home/OraDB12c/ 
$ unzip linuxamd64_12c_database_1of2.zip
$ unzip linuxamd64_12c_database_2of2.zip

```

## Add Permission for Oracle User to xhost

Next, run the following command as root to avoid a common display error during the Oracle 11g installation.

```shell
# xhost +SI:localuser:oracle 
  localuser:oracle being added to access control list
```

Edit `/home/oracle/.bash_profile` with your preferred editor and add the following environment variables (substituting your hostname and other installation preferences).

```shell
# Oracle Settings
TMP=/tmp; export TMP
TMPDIR=$TMP; export TMPDIR

ORACLE_HOSTNAME=gps12c.localdomain
ORACLE_UNQNAME=orcl
ORACLE_BASE=/home/oracle/app/oracle
ORACLE_HOME=/home/oracle/app/oracle/product/12.1.0/dbhome_1
ORACLE_SID=orcl
export ORACLE_SID ORACLE_HOME ORACLE_BASE ORACLE_UNQNAME ORACLE_HOSTNAME
PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH
export PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib;
export CLASSPATH LD_LIBRARY_PATH
```

Switch to the Oracle user and change to the OraDB12c directory to run the installation.

```shell
# su - oracle
# cd /home/OraDB12c/database
# ./runInstaller
```

Since mine is a home machine, I accepted the default values on most screens. I've included all the screenshots below.

Leave the first page blank unless you have an Oracle Support account.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_2.png" description="Permission Error" %}

This page is different from 11g. I skipped the software updates since I had none to install.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_3.png" description="Download Software Updates" %}

This option will launch the database configuration assistant after the software is installed.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_4.png" description="Select Installation Options" %}

A desktop class is sufficient for a home spatial learning lab.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_5.png" description="Pick System Class" %}

Be sure to deselect "Create as Container database" unless you need to manage PDB (Pluggable Databases). 

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/step5of9.png" description="Deselect Create Container Database" %}

Take the default inventory directory.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_7.png" description="Create Inventory" %}

With the three pre-install steps noted above, my system passed all the prerequisite checks and skipped directly to the summary page.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_8.png" description="Installation Summary" %}

Run the two scripts shown as root.

{% include image.html url="/img/post-assets/2013-06-25-installing-oracle-spatial-and-graph-12c-on-OEL-6.4/12c_9.png" description="Run Installation Scripts" %}

## Test the Oracle Installation

Try connecting to the database as the user Oracle using the commands shown below. Run the test query shown to pull back the database SID.

```shell
$ sqlplus / as sysdba

SQL*Plus: Release 12.1.0.1.0 Production on Tue Jun 25 22:01:38 2013
Copyright (c) 1982, 2013, Oracle. All rights reserved.

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.1.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics, and Real Application Testing options

SQL> select instance_name from v$instance;

INSTANCE_NAME
----------------
orcl

SQL> exit

Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.1.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics, and Real Application Testing options.
```

## Wrapping Up

That is good enough for tonight. In my future posts, I'll use 12c instead of 11g, even though it will probably be a long time before we upgrade at work. I want access to the latest spatial features as I am learning.
