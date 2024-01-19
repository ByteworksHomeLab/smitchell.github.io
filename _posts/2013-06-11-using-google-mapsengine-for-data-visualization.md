---
layout: post
title:  "Using Google MapsEngine for Data Visualization"
url: /using-google-mapsengine-for-data-visualization
comments: true
date: 2013-06-11 7:25:00
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
<a href="./using-google-mapsengine-for-data-visualization">
    <img 
        src="/img/post-assets/2013-06-11-using-google-mapsengine-for-data-visualization/mapsenginelogo.png" 
        alt="Maps Engine Logo"
    >
</a>


Yesterday, I began Google's two-week online course Mapping with Google. I highly recommend taking it if you have time. The new and enhanced features of the Google Map beta are impressive.
Lesson 2 of Mapping with Google is on Google's Maps Engine. It is a handy tool for creating and sharing simple maps. The first thing I did when I got home last night was create a map of our upcoming Washington, D.C. vacation. I added points of interest for our hotel, some museums, a university we want my daughter to tour, and some areas I would like to see, like Chinatown. I shared the map with my wife and daughters and gave them all edit access. It will be interesting to see if they start making changes today.

It is helpful to visualize spatial data to see that the element coordinates are correctly defined, so I imported the test data from my post Getting Started with Oracle Spatial using SQL*PLUS into Google Maps Engine. I couldn't import the track's polygon, so I manually drew it. The points in the parking lot and infield were imported fine. As you can see below, the "Kansas City Speedway Infield" point, marked with the wrench icon, appears where it should be in the track infield. The "Kansas Speedway Parking" point, the orange "P" icon, appears where it should be in the parking area.

In an upcoming post, I will demonstrate the same thing in Oracle MapViewer, reading directly from the table in the database.

In the meantime, here is the Google Maps Engine version link.

{% include image.html url="/img/post-assets/2013-06-11-using-google-mapsengine-for-data-visualization/mapsengine.png" description="" %}
