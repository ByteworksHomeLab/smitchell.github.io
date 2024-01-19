---
layout: post
title:  "Installing MapBox TileMill on Ubuntu"
url: /installing-mapbox-tilemill-on-ubuntu
comments: true
date: 2013-06-20 20:44:00
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
<a href="./installing-mapbox-tilemill-on-ubuntu">
    <img 
        src="/img/post-assets/2013-06-20-installing-mapbox-tilemill-on-ubuntu/tilemill_logo.jpg" 
        alt="TileMill Logo"
    >
</a>
# Preparing to Work with Satellite Imagery

I'm a Java developer, not a GIS imagery specialist, but when I saw the post "Processing Landsat 8 Using Open-Source Tools" by Charlie Loyd, I just had to try it. Charlie's post is a step-by-step tutorial on how to manipulate Landsat 8 satellite images.

There are four prerequisites:
1. GDAL, a low-level GIS toolkit
2. libgeotiff, to work with geotags
3. ImageMagick, an image-processing package
4. TileMill, an open-source mapmaking app from Mapbox

This post walks through the installation of these four tools.

## 1) Install GDAL

I followed this install-gdal script, except I executed the commands individually to know what the script was doing.
```shell
$ sudo apt-get -y install g++
$ svn checkout https://svn.osgeo.org/gdal/trunk/gdal gdal
$ cd gdal
$ ./configure
$ sudo make install
...long build process...
libtool: install: /home/smitchell/gdal/install-sh -c .libs/gdalbuildvrt /usr/local/bin/gdalbuildvrt
/bin/bash /home/smitchell/gdal/libtool --mode=install  /home/smitchell/gdal/install-sh -c gdal-config-inst /usr/local/bin/gdal-config
libtool: install: /home/smitchell/gdal/install-sh -c gdal-config-inst /usr/local/bin/gdal-config
make[1]: Leaving directory `/home/smitchell/gdal/apps'
for f in LICENSE.TXT data/*.* ; do /home/smitchell/gdal/install-sh -c -m 0644 $f /usr/local/share/gdal ; done
/bin/bash /home/smitchell/gdal/libtool --mode=finish --silent /usr/local/lib
```

After GDAL is installed you need to change your LD_LIBRARY_PATH. Edit $HOME/.bashrc and add the following line:

```shell
LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH
```

## 2) Install libgeotiff

Libgeotiff was the only tool I found in the Ubuntu Software Center, although I wasn't sure which one to install since I'm running 64-bit Ubuntu. I went with libgeotiff2.

{% include image.html url="/img/post-assets/2013-06-20-installing-mapbox-tilemill-on-ubuntu/libgeotiff.png" description="The GeoTIFF Library" %}

# 3) Install Imagemagick

Install Imagemagick and imagick:

```shell
$ sudo apt-get install imagemagick php5-imagick
```

# 4) Install TileMill.

Finally, we are ready to install TileMill.

```shell
sudo add-apt-repository ppa:developmentseed/mapbox
sudo apt-get update
sudo apt-get install tilemill libmapnik nodejs
```

Start TileMill from the Launcher once TileMill is installed.

{% include image.html url="/img/post-assets/2013-06-20-installing-mapbox-tilemill-on-ubuntu/linux-install-4.png" description="Launch TileMill" %}

That's it, you installed TileMill. 

{% include image.html url="/img/post-assets/2013-06-20-installing-mapbox-tilemill-on-ubuntu/tilemill.png" description="TileMill Page" %}

For my next post, I will grab interesting images and follow Charlie's tutorial.
