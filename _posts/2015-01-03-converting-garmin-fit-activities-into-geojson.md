---
layout: post
title:  "Converting Garmin FIT activities into GeoJSON"
url: /converting-garmin-fit-activities-into-geojson
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
<a href="./converting-garmin-fit-activities-into-geojson">
    <img 
        src="/img/post-assets/2015-01-03-converting-garmin-fit-activities-into-geojson/LittleRockMarathon.png" 
        alt="Little Rock Marathon"
    >
</a>

After creating a Leaflet demo using Bing, OSM, and Google base maps, the next step is to display the activity polyline as a Leaflet layer instead of adding it directly to Google, Bing, or OSM maps via the vendors' map APIs. With Leaflet, the base map is simply the background for the vector graphics displayed on a Leaflet map layer in the foreground.

## Using GeoJSON with Leaflet

First, I needed to produce [GeoJSON](https://leafletjs.com/examples/geojson.html). GeoJSON is a JSON [standard for geometry types](https://geojson.org). The geometry types are:

* Point
* LineString
* Polygon
* MultiPoint
* MultiLineString
* MultiPolygon

Geometry types may be combined with additional properties using Feature or FeatureCollection. Leaflet has a map layer type that can consume Feature JSON or FeatureCollection JSON.

## Garmin Activity FIT File in --> GeoJSON out

Modern Garmin fitness devices store data in a compressed binary format called the [FIT protocol]( https://thisisant.com). My first task was to create a utility to read Garmin activity FIT files and produce Feature GeoJSON. The code is in my [`garmin-fit-geojson` repository on GitHub](https://github.com/smitchell/garmin-fit-geojson).

I used my [2012 Little Rock Marathon activity](https://connect.garmin.com/modern/activity/155155867) to test the GeoJSON utility. The FIT SDK decodes the activity FIT file. The GeoTools FeatureJSON class outputs the GeoJSON.

There are two steps to creating GeoJSON using the GeoTools API:

1. Define a feature type (schema) for the feature properties.
2. Build the feature following the feature type definition.

## Defining a Simple Feature Type

The SimpleFeatureTypeBuilder class is used to layout the schema for the Feature

```java
public SimpleFeatureType getFeatureSchema() {
    final SimpleFeatureTypeBuilder simpleFeatureType = new SimpleFeatureTypeBuilder();
    simpleFeatureType.add("geom", LineString.class, DefaultGeographicCRS.WGS84);
    simpleFeatureType.add("name", String.class);
    simpleFeatureType.add("activityId", Long.class);
    simpleFeatureType.setName("activity");
    simpleFeatureType.add("activityName", String.class);
    simpleFeatureType.add("sport", String.class);
    simpleFeatureType.add("startTime", String.class);
    simpleFeatureType.add("totalMeters", Double.class);
    simpleFeatureType.add("totalSeconds", Double.class);
    simpleFeatureType.add("minLat", Double.class);
    simpleFeatureType.add("minLon", Double.class);
    simpleFeatureType.add("maxLat", Double.class);
    simpleFeatureType.add("maxLon", Double.class);
    return simpleFeatureType.buildFeatureType();
}
```

## Building the Feature

The SimpleFeatureBuilder builds the SimpleFeature following the feature schema definition.

```java
public SimpleFeature buildSimpleFeature(final FitActivity fitActivity) {
    final SimpleFeatureType featureSchema = getFeatureSchema();
    final SimpleFeatureBuilder builder = new SimpleFeatureBuilder(featureSchema);
    builder.set("activityId", fitActivity.getActivityId());
    builder.set("sport", fitActivity.getSport());
    builder.set("startTime", fitActivity.getStartTime());
    builder.set("totalMeters", fitActivity.getTotalMeters());
    builder.set("totalSeconds", fitActivity.getTotalSeconds());
    final Coordinate[] polyline = fitActivity.getPolyline().toArray(
        new Coordinate[fitActivity.getPolyline().size()]);
    final Geometry geometry = simplifyLineString(polyline);
    builder.add(geometry);
    final Coordinate[] boundingBox = generateBoundingBox(geometry);
    builder.set("minLat", boundingBox[0].y);
    builder.set("minLon", boundingBox[0].x);
    builder.set("maxLat", boundingBox[1].y);
    builder.set("maxLon", boundingBox[1].x);
    return builder.buildFeature("0");
}
```

The final result, formatted for readability, can be found here: [feature.json](https://github.com/smitchell/garmin-fit-geojson/blob/master/src/test/resources/feature.json)

This is a partial listing:

```json
{
  "type": "Feature",
  "geometry": {
    "type": "LineString",
    "coordinates": [
      [
        -92.2639,
        34.7473
      ],

      [
        -92.2668,
        34.7484
      ]
    ]
  },
  "properties": {
    "activityId": 155155867,
    "sport": "RUNNING",
    "startTime": "2012-03-04T14:02Z",
    "totalMeters": 42453.58984375,
    "totalSeconds": 15162.140625,
    "minLat": 34.73176879808307,
    "minLon": -92.34434505924582,
    "maxLat": 34.78602569550276,
    "maxLon": -92.25817699916661
  },
  "id": "0"
}
```

Now I'm ready to create a Leaflet GeoJSON layer to display the polyline!
