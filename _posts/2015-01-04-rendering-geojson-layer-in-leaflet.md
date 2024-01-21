---
layout: post
title:  "Rendering a GeoJSON layer in Leaflet"
url: /rendering-geojson-layer-in-leaflet
comments: true
date: 2015-01-04 19:30:00
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
<a href="./rendering-geojson-layer-in-leaflet">
    <img 
        src="/img/post-assets/2015-01-04-rendering-geojson-layer-in-leaflet/demo4.png" 
        alt="GeoJSON on Leaflet"
    >
</a>

Adding a GeoJSON layer showing a Garmin activity polyline completes my example of using Leaflet with Bing, Google, and OSM base layers.

You can find my code for a [static GeoJSON Leaflet map with Google, Bing, and OSM base layers is in my GitHub](https://github.com/smitchell/exploringspatial).

I did a lot of work to get to this point. Some tasks were unrelated to the demo, like converting my website to a single-page Backbone application. Other tasks were behind the scenes, like writing a utility to convert Garmin FIT files into GeoJSON. Much of my time was spent recreating the Garmin-style map controls to switch between Google, Bing, and OSM. Eventually, I want to add the Google bicycle path layer and fix a bug I found today on the menu controls, but that can wait.

Coding was a breeze once I was ready to add the polyline and custom pin icons. All that was needed was to extend L.Icon for the custom pin icons and drop the polyline GeoJSON into the L.geoJson function, as shown below:


```javascript
    var ActivityMapLayerView = Backbone.View.extend({
        initialize: function(args) {
            this.map = args.map;
            var CustomIcon = L.Icon.extend({options: {
                iconSize: [33, 50],
                iconAnchor: [16, 49]
            }});
            this.startIcon = new CustomIcon({iconUrl: 'media/pin_start.png'});
            this.endIcon = new CustomIcon({iconUrl: 'media/pin_end.png'});
            this.render();
        },

        render: function() {
            var props = this.model.get('properties');
            this.map.fitBounds([
                [props.get('minLat'), props.get('minLon')],
                [props.get('maxLat'), props.get('maxLon')]
            ]);
            var style = {
                color: '#FF0000',
                weight: 3,
                opacity: 0.6
            };
            L.geoJson(this.model.toJSON(), {style: style}).addTo(this.map);
            var polyline = this.model.get('geometry').get('coordinates');
            var startPoint = polyline[0];
            var endPoint = polyline[polyline.length - 1];
            L.marker([startPoint[1], startPoint[0]], {icon: this.startIcon}).addTo(this.map);
            L.marker([endPoint[1], endPoint[0]], {icon: this.endIcon}).addTo(this.map);
        }

    });
```

The GeoJSON is passed into L.geoJson along with style properties to control the polyline's color, weight, and opacity. That is all it took!

It is time to begin exploring the plethora of Leaflet plugins supporting interactive maps. I'm looking forward to getting started.
