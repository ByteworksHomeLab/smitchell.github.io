---
layout: post
title:  "Getting Started with Oracle Spatial using SQLPLUS"
url: /getting-started-with-oracle-spatial-using-SQLPLUS
comments: true
date: 2013-06-08 9:27:00
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
{% include image.html url="/img/post-assets/2013-06-13-getting-started-with-oracle-spatial-using-SQLPLUS/sqlplus.gif" description="" %}

In my previous post, How to Install Oracle Spatial and Graph on OEL 6.4, I showed how to install Oracle 11g Enterprise Edition. In this post, I show how to use SQL*PLUS to create a test user account and a spatially indexed table into which the path of the Kansas Speedway is inserted. Finally, I show a couple of straightforward spatial queries.

{% include image.html url="/img/post-assets/2013-06-13-getting-started-with-oracle-spatial-using-SQLPLUS/googlemapsengine.png" description="Kansas Speedway" %}

## Start the Database

Recapping from my last post, you should now be able to start your Oracle 11g EE database using SQL*PLUS.

```
$ su - oracle
Password:
$ sqlplus sys as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on Mon Jun 3 19:59:35 2013
Copyright (c) 1982, 2009, Oracle. All rights reserved.
Enter password:
Connected to an idle instance.

SQL> startup
ORACLE instance started.
```
## Create a New Oracle User

Before signing out of SQL*PLUS, create a new Oracle user account. Here, you see that I created a user named STEVE and granted that account the privileges of DBA, CONNECT, and RESOURCE.

```
SQL> GRANT DBA, CONNECT, RESOURCE TO STEVE IDENTIFIED BY nHMdfGcg0aTt;
Grant succeeded.
SQL> exit
Exit SQL*PLUS after creating the new user.
```

## Start the TNS Listener

The TNS listener must be started before signing in as a new user. From the Oracle Linux account, execute the following command:

```
lsnrctl start
```

When the listener begins, its output will look something like this:

```text
LSNRCTL for Linux: Version 11.2.0.1.0 - Production on 07-JUN-2013 17:59:40

Copyright (c) 1991, 2009, Oracle.  All rights reserved.

Starting /home/oracle/app/oracle/product/11.2.0/dbhome_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 11.2.0.1.0 - Production
System parameter file is /home/oracle/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Log messages written to /home/oracle/app/oracle/diag/tnslsnr/gps11g/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=gps11g)(PORT=1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.1.0 - Production
Start Date                07-JUN-2013 17:59:42
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /home/oracle/app/oracle/product/11.2.0/dbhome_1/network/admin/listener.ora
Listener Log File         /home/oracle/app/oracle/diag/tnslsnr/gps11g/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=gps11g)(PORT=1521)))
The listener supports no services
The command completed successfully
```

## Create and Index the Spatial Table

For this post, all that is needed is a straightforward table. There is no need to won't worry about primary keys, sequences, or constraints.

The SQL below does three things:

1. Defines a table containing a spatial column.
2. Insert spatial metadata for the table.
3. Creates a spatial index on the spatial column.

Sign in using the new user account and run the SQL shown below.

```text
$ sqlplus STEVE@orcl

SQL*Plus: Release 11.2.0.1.0 Production on Fri Jun 7 18:08:22 2013

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Enter password: 

Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> CREATE TABLE SPATIAL_LEARNING (
    SPATIAL_NAME VARCHAR2(50) NOT NULL,
    SPATIAL_DATA SDO_GEOMETRY NOT NULL);
Table created.

SQL> INSERT INTO USER_SDO_GEOM_METADATA
    (TABLE_NAME,
     COLUMN_NAME,
     DIMINFO,
     SRID)
  VALUES (
  'SPATIAL_LEARNING',
  'SPATIAL_DATA',
  SDO_DIM_ARRAY( 
    mdsys.sdo_dim_element('longitude',-180,180,0.05), 
    mdsys.sdo_dim_element ('latitude',-90,90,0.05)
     ),
  8307
);  
1 row created.

SQL> CREATE INDEX SPATIAL_LEARNING_DATA_IDX
ON SPATIAL_LEARNING (SPATIAL_DATA) 
INDEXTYPE IS MDSYS.SPATIAL_INDEX;  
Index created.
```
Let's review the code above.

### Create Table with SDO_GEOMETRY Data Type

The SPATIAL_DATA column of the SPATIAL_LEARNING table is defined with the SDO_GEOMETRY data type. That's the column where the spatial data will be stored. The SDO_GEOMETRY type is discussed in more detail below.

### Spatial Metadata

Spatial metadata is defined in the USER_SDO_GEOM_METADATA table. The metadata describes the dimensions, upper and lower bounds, and the tolerance of the spatial data. In this case, we declare that the SPATIAL_DATA column has two dimensions: "latitude" and "longitude." The upper and lower bounds of latitude are -90 through 90. The upper and lower bounds of longitude are -180 through 180. The points stored in the SPATIAL_DATA column have a tolerance of 0.05, meaning they can be as close as 5 cm apart. Finally, the SRID (Spatial Reference System Identifier) is 8307.

### Spatial Index

Once the spatial metadata is defined, an index may be created using the MDSYS.SPATIAL_INDEX type.

## Understanding SDO_GEOMETRY

Let's get familiar with the SDO_GEOMETRY type before inserting the data. Here is its definition which shows its component parts.

```
CREATE TYPE sdo_geometry AS OBJECT ( 
  SDO_GTYPE NUMBER, 
  SDO_SRID NUMBER, 
  SDO_POINT SDO_POINT_TYPE, 
  SDO_ELEM_INFO SDO_ELEM_INFO_ARRAY, 
  SDO_ORDINATES SDO_ORDINATE_ARRAY
);
```

See the Spatial Data Types and Metadata section on SDO_GEOMETRY for complete reference information. Let's take a moment to go over the SDO_GEOMETRY information pertinent to the test data used in this post.

### SDO_GTYPE

The SDO_GTYPE tells oracle what kind of data the SDO_GEOMETRY object contains. The test data includes 2003, polygon, and 2001, point.

Here are some of the SDO_GTYPES:

* Point: 2001 (2D), 3001 (3D)
* Line : 2002 (2D), 3002 (3D)
* Polygon: 2003 (2D), 3003 (3D)
* Collection: 2004 (2D), 3004 (3D)
* Multipoint: 2005(2D), 3005 (3D)
* Multiline: 2006(2D), 3006 (3D)
* Multipolyline: 2007(2D), 3006 (3D)

### SDO_SRID

The SDO_SRID (Spatial Reference System Identifier) for the test data is 8307, the WGS-84 type for a latitude and longitude coordinate system. More about SRID types can be found in the introduction to coordinate reference systems.

### SDO_POINT

Single points can be declared with the SDO_POINT_TYPE, in which case the remaining attributes of SDO_GEOMETRY are left null. Do not use the SDO_POINT_TYPE if you use Oracle LRS (Linear Reference System). The SDO_POINT_TYPE is used below for the points in the speedway infield and the parking lot. SDO_POINT is null for the speedway track polygon because it uses SDO_ELEMENT_INFO and SDO_ORDINATES instead.

### SDO_ELEMENT_INFO

To use SDO_ORDINATES, set SDO_POINT to null and define SDO_ELEM_INFO. SDO_ELEM_INFO is the metadata for the elements in the subsequent SDO_ORDINATES array. The SDO_ELEM_INFO is declared using  SDO_ELEM_INFO_ARRAY. It takes three arguments:

1. **Ordinate Starting Offset** - The offset into the ordinate array at which to start.
2. **Element Type** - The type of ordinates. For example: 
  * 1003: exterior polygon ring (must be specified in counterclockwise order). 
  * 2003: interior polygon ring (must be specified in clockwise order)
3. **Interpretation** - If the element type is a compound type, this column specifies how many subsequent triplet values are part of the element. Otherwise, it specifies whether to connect the ordinates by straight lines or arcs.

SDO_ORDINATES - The SDO_ORDINATES is an array of coordinate values. For a 2D line, the array would contain "long,lat,long, lat,, ..." or a 3D SRID the array would contain (long,lat, elevation,long,lat, elevation, ...). Note there is no delimiter separating ordinates for one element from another. That is why the preceding element info declaration is so important. It tells Oracle how to group the ordinates together into elements.

### Insert the Spatial Test Data

You are now ready to insert three rows into our SPATIAL_LEARNING table:

1) An interior polygon ring tracing the track at the Kansas Speedway.
2) A single point in the infield.
3) A single point in a parking lot outside the speedway.

## Insert the Speedway Track Polygon

Use the following insert statement to store the polygon representing the oval track at the Kansas Speedway. 1003 in the SDO_ELEM_INFO_ARRAY indicates an exterior polygon ring and the 1 way to connect the elements with straight lines. Also note that the first and last points are identical, thus closing the ring.

```text
SQL> INSERT INTO SPATIAL_LEARNING 
(SPATIAL_NAME, SPATIAL_DATA)
VALUES
( 'Kansas Speedway Track',
  SDO_GEOMETRY(
    2003, -- two-dimensional polygon
    8307, -- SRID for WGS-84 longitude/latitude 
    NULL,
    SDO_ELEM_INFO_ARRAY(1, 1003, 1),
    SDO_ORDINATE_ARRAY(
        -94.8338234424591,39.11655560213989,
        -94.83394682407379,39.11618478057385,
        -94.83402192592621,39.11574920989278,
        -94.83404874801636,39.11532540880791,
        -94.83400046825409,39.114819198615976,
        -94.83385562896729,39.11417171516305,
        -94.83353912830353,39.11296502557993,
        -94.83324944972992,39.11248823034213,
        -94.8328846693039,39.11213504660467,
        -94.83245015144348,39.11187604407223,
        -94.83188152313232,39.1116700186508,
        -94.8313558101654,39.1115581760269,
        -94.8308676481247,39.11156994894302,
        -94.83044385910034,39.111611154134025,
        -94.82995569705963,39.111711223783246,
        -94.82944071292877,39.111934908367736,
        -94.82903838157654,39.11224688831305,
        -94.82870578765869,39.11257652599992,
        -94.82847511768341,39.11292382118056,
        -94.82679605484009,39.11707357185418,
        -94.82670485973358,39.117479704535015,
        -94.82674241065979,39.118021211134604,
        -94.8269248008728,39.11850974069151,
        -94.82704818248749,39.118739289554505,
        -94.82735931873322,39.119121869330876,
        -94.82782065868378,39.119427931656176,
        -94.82835710048676,39.11965747752762,
        -94.8288881778717,39.119781078840944,
        -94.82943534851074,39.11981639346204,
        -94.82993960380554,39.11977519306902,
        -94.83050286769867,39.11964570596266,
        -94.83092129230499,39.119422045854755,
        -94.83125925064087,39.11921015667656,
        -94.83244478702545,39.11835670770069,
        -94.8330295085907,39.11783874741924,
        -94.83343183994293,39.11730901137464,
        -94.83371078968048,39.11684401756424,
        -94.8338234424591,39.11655560213989
     ) 
  ) 
); 
1 row created.
```
### Insert a point in the Speedway Parking Lot

Next use the SDO_POINT_TYPE to define a nearby point in the speedway parking lot.

```test
INSERT INTO SPATIAL_LEARNING (SPATIAL_NAME, SPATIAL_DATA) VALUES
( 'Kansas Speedway Infield',
  SDO_GEOMETRY( 2001,
    8307,
    SDO_POINT_TYPE(-94.8312053, 39.1153637, null),
    null,
    null
  )
);
```
## Run a Simple Spatial Query

You are now ready to query the spatial data. These examples use SDO_FILTER to select geometry falling within the bounds of a rectangular specified by the lower left and upper correct coordinates. The first rectangle is a park in Monterrey, Mexico. The second rectangle encloses the speedway. Running the first query should not return any result. The second query returns all three rows of the SPATIAL_LEARNING table.

```text 
SQL> SELECT t1.SPATIAL_NAME
FROM SPATIAL_LEARNING t1
WHERE SDO_FILTER(
  t1.SPATIAL_DATA,
  SDO_GEOMETRY(
    2001, 8307, NULL,
    DO_ELEM_INFO_ARRAY(1,1003,3),
    SDO_ORDINATE_ARRAY(
      -100.347404, 25.762179,
      -100.336082, 25.775009
    )
  )
) = 'TRUE';

no rows selected

SQL> SELECT t1.SPATIAL_NAME
FROM SPATIAL_LEARNING t1
WHERE SDO_FILTER(
  t1.SPATIAL_DATA,
  SDO_GEOMETRY(
    2001, 8307, NULL,
    SDO_ELEM_INFO_ARRAY(1,1003,3),
    SDO_ORDINATE_ARRAY(
      -94.852695, 39.106554,
      -94.816046, 39.124068
    )
  )
) = 'TRUE';

SPATIAL_NAME
--------------------------------------------------
Kansas Speedway Track
Kansas Speedway Infield
Kansas Speedway Parking
```

One mystery remains: the following query is an SDO_FILTER using a rectangle enclosing the infield track point. The southwest and northeast corners of the rectangle are marked on the Google MapsEngine map pictured at the beginning of this post. Click on the map image and open the map in Google MapsEngine.

I expected it to return only the infield point, but it also returns the track's polygon, perhaps because the rectangle is in the interior area of the polygon. I'll try to track down the answer.

```text
SQL> SELECT t1.SPATIAL_NAME 
FROM SPATIAL_LEARNING t1 
WHERE SDO_FILTER(
  t1.SPATIAL_DATA,
  SDO_GEOMETRY(
   2001, 8307, NULL,
   SDO_ELEM_INFO_ARRAY(1,1003,3),
   SDO_ORDINATE_ARRAY(
     -94.832375, 39.1146286, 
     -94.8304224, 39.1158106
   )
  )
) = 'TRUE';

SPATIAL_NAME
--------------------------------------------------
Kansas Speedway Track
Kansas Speedway Infield
```

There are many spatial operators and functions to be explored in future posts, but our next post will discuss installing additional Oracle tools: SQL Developer and Map Viewer.
