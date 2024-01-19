---
layout: post
title:  "How to Install Oracle MapViewer"
url: /how-to-install-oracle-mapviewer
comments: true
date: 2013-06-09 21:31:00
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
<a href="./how-to-install-oracle-mapviewer">
    <img 
        src="/img/post-assets/2013-06-09-how-to-install-oracle-mapviewer/mapviewerlogo.png" 
        alt="MapViewer Logo"
    >
</a>
This post describes installing Oracle MapViewer 11g (11.1.1.7) Quick Start Kit. We'll also discuss tracking down MapViewer demos not included with this distribution. You'll walk through the import of that data step-by-step and the creation of a permanent data source.

## Download Oracle MapViewer

Go to the Software Downloads for Oracle Fusion Middleware MapViewer page and accept the Terms and Conditions. Download the MapViewer Quick Start Kit from the Current MapViewer Version section. The Quick Start kit is bundled with Glassfish server version 3.1.2. Alternatively, you can download and unzip the EAR file and then deploy the WAR file from the EAR to Tomcat.
Unzip the file and read the README.txt file located in the mapviewer11g_gs directory.

## Install and Configure Glassfish and MapViewer

Installation is simply a matter of following the instructions in the README.txt file.

```shell
$ ./runMeFirst.sh
Setting 'admin' user's password to: welcome1 ...
Command change-admin-password executed successfully.
Waiting for domain1 to start ......
Successfully started the domain : domain1
domain  Location: /home/smitchell/Applications/mapviewer11g_qs/glassfish3/glassfish/domains/domain1
Log File: /home/smitchell/Applications/mapviewer11g_qs/glassfish3/glassfish/domains/domain1/logs/server.log
Admin Port: 4848
Command start-domain executed successfully.
Deploying Oracle MapViewer ......
Application deployed with name mapviewer.
Command deploy executed successfully.
Deploying MapViewer Samples application ......
Application deployed with name mvdemo.
Command deploy executed successfully.
Enabling remote secure login
You must restart all running servers for the change in secure admin to take effect.
Command enable-secure-admin executed successfully.
Restarting server ...
Successfully restarted the domain
Command restart-domain executed successfully.
MapViewer QuickStart kit is now running!
MapViewer Server URL: http://localhost:8080/mapviewer
MapViewer Samples URL: http://localhost:8080/mvdemo
Glassfish Admin URL:  http://localhost:4848
```

Next, remove the one-time-use files used by runMeFirst.sh.

```shell
rm welcome1.txt
rm password.txt
```

MapViewer is now running. To start and stop Glassfish/MapViewer run the startServer and stopServer scripts.

## Configure the MapViewer Demo

The MapViewer Quick Start Kit comes bundled with the MVDEMO war. Some housekeeping is necessary to set up the demo. That is all explained on the MVDEMO homepage: http://localhost:8080/mvdemo/.

## Confirm the Required Database Views Exist

Oracle Enterprise Edition creates multiple views in MDSYS. Open SQL Developer and connect to the database. Scroll down to Other Users. Expand Other Users and find the user MDSYS. Expand the Views folder and scroll down until you see views starting with "USER_SDO_...". Those views should include `USER_SDO_CACHED_MAPS`, `USER_SDO_MAPS`, `USER_SDO_THEMES`, and `USER_SDO_STYLES`.

{% include image.html url="/img/post-assets/2013-06-09-how-to-install-oracle-mapviewer/mdsys_views.png" description="Spatial Lab" %}

#  Import the MapViewer Demo Data

A file named `mvdemo.zip` contains the data for the MapViewer demo. It isn't part of the `maviewer11g_qs` download. I found a copy in `mapviewer10133wls.zip`. The `mvdemo.zip` file contains four SQL files, a `mvdemo.dmp` file, and `readme.txt`.
Follow the instructions in the `readme.txt`file found in `mvdemo.zip` file. It walks you through the following steps.
## Create the Oracle User
Create a new user, mvdemo. You'll need that user to import the demo data later, although you may use a different user if you wish.

```oraclesqlplus
grant connect, resource, create view to mvdemo identified by mvdemo;
Grant succeeded.
```

## Import the Dump Data
I'm working on two machines, so for this step, I used Secure Copy to copy the dump data to the Oracle database server.

```shell
scp mvdemo.dmp oracle@gps11g:~/ 
oracle@gps11g's password: 
mvdemo.dmp 100% 8288KB 8.1MB/s 00:00
```

I ran the import command from the oracle user account on the database server.

```shell
$ imp mvdemo/mvdemo file=mvdemo.dmp full=y ignore=y

Import: Release 11.2.0.1.0 - Production on Sun Jun 9 19:43:47 2013
Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

Export file created by EXPORT:V09.00.01 via conventional path
import done in US7ASCII character set and AL16UTF16 NCHAR character set
import server uses WE8MSWIN1252 character set (possible charset conversion)
export client uses WE8MSWIN1252 character set (possible charset conversion)
. importing MVDEMO's objects into MVDEMO
. importing MVDEMO's objects into MVDEMO
. . importing table                       "CITIES"        195 rows imported
. . importing table                     "COUNTIES"       3230 rows imported
. . importing table                    "EMPLOYEES"         14 rows imported
. . importing table                  "INTERSTATES"        239 rows imported
. . importing table                         "MAPS"          4 rows imported
. . importing table                       "STATES"         56 rows imported
. . importing table                       "STYLES"        285 rows imported
. . importing table                  "TERRITORIES"          9 rows imported
. . importing table                "TERR_COUNTIES"       3230 rows imported
. . importing table                       "THEMES"         10 rows imported
Import terminated successfully without warnings.
Run mvdemo.sql
```

The Readme.txt file suggests importing mvdemo.sql from SQL*PLUS; however, I did it from SQL Developer using a new connection authenticated with the new `mvdemo` user account.

{% include image.html url="/img/post-assets/2013-06-09-how-to-install-oracle-mapviewer/sqldemo_import.png" description="Spatial Demo Import" %}

That completes the mvdemo data import. 

## Create the mvdemo Data Source

Open the MapViewer Administrative console, http://localhost:8080/mapviewer/, and click the Admin link circled below.

{% include image.html url="/img/post-assets/2013-06-09-how-to-install-oracle-mapviewer/mapvieweradmin.png" description="Map Viewer Admin" %}

Follow these steps:

1. Sign in with the Glassfish ID and password, admin/welcome1. You may change the Glassfish password by running the resetPassword.sh script.
2. When the Management page appears, click Configuration at the top of the menu on the left.
3. The MapperConfig file will appear. Scroll to the bottom of the file to the Predefined Data Sources section.
4. Un-comment the mvdemo map_data_source and edit the values to match the database server as shown below.NOTE: the exclamation point before the password is not a typo. It must be there.
5. Click the Save and Restart button at the bottom of the Configuration page.

Click on the Data Sources menu below the Configuration menu after the MapViewer restarts, and you'll see the new data source.

{% include image.html url="/img/post-assets/2013-06-09-how-to-install-oracle-mapviewer/mvdemo_ds.png" description="MVDemo Data Source" %}

## Explore the Demos

You can now start exploring the demos at http://localhost:8080/mvdemo/demo/oracle_maps_demo.jsp.

{% include image.html url="/img/post-assets/2013-06-09-how-to-install-oracle-mapviewer/demopage.png" description="Demo Page" %}

In my next post I'll set up the HTML5 demos, that is, once I've finished exploring this demo set.


