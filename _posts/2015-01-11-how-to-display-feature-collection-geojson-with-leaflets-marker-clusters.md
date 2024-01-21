---
layout: post
title:  "How to Display Feature Collection GeoJson with Leaflet's Marker Clusters"
url: /how-to-display-feature-collection-geojson-with-leaflets-marker-clusters
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
<a href="./how-to-display-feature-collection-geojson-with-leaflets-marker-clusters">
    <img 
        src="/img/post-assets/2015-01-11-how-to-display-feature-collection-geojson-with-leaflets-marker-clusters/screen-shot-2015-01-11-at-4-58-56-pm.png" 
        alt="GeoJSON on Leaflet"
    >
</a>

I enjoyed playing with Leaflet's marker cluster plugin this weekend while writing a new demonstration that shows how to load feature collection GeoJson into a Leaflet map layer.

It was demo was surprisingly easy to set up:

1. I updated my [GeoJson utility project](https://exploringspatial.wordpress.com/2015/01/03/converting-garmin-fit-activities-into-geojson/) to create a Feature Collection GeoJson loaded with [768 of my runs from Garmin Connect](http://www.exploringspatial.com/activities). The features in the feature collection are summaries. They contain the same properties as the activity detail GeoJson from my previous demo, except that they use a starting latitude/longitude instead of a polyline.
2. I generated individual feature GeoJson files with the full polyline for each run.
3. I added the new GeoJson files to my website at http://www.exploringspatial.com/activity/.
4. A Backbone Collection, [Activities.js](https://github.com/smitchell/exploringspatial/blob/master/js/collections/Activities.js), uses AJAX to fetch the activity summaries. The collection is passed into the ActivitiesMapLayerView.js. The code in its render function is simple:

```javascript
render: function() {
    var _self = this;
    geoJsonLayer = L.geoJson(this.collection.toJSON(),{
        onEachFeature: _self.onEachFeature
    });
    this.map.fitBounds(geojson.getBounds());
    this.activitiesLayer = L.markerClusterGroup();
    this.activitiesLayer.addLayer(geoJsonLayer);
    this.map.addLayer(this.activitiesLayer);
    this.map.on('popupopen', function(event) {_self.onPopupOpen(event);});
    $('.returnToSearch').on('click', '.returnTrigger', function(event){
        _self.onReturnToSearch(event)
      });
}
```

1. L.geoJson - Create a map layer by passing in the activity collection JSON and an onEachFeature function (see below).
2. L.markerClusterGroup - Create a marker cluster map layer.
3. addLayer - Add the GeoJson layer to the marker cluster group.
4. addLayer - Add the marker cluster group to the map.
5. 
The last two lines of code bind a listener to the Popupopen event and add a listener for the activity detail page event that returns to the main page.
The onEachFeature function binds a popup to each feature marker on the map. It does date and number formatting and then passes HTML into layer.bindPopup.

```javascript
onEachFeature: function(feature, layer) {
    var date = new Date(feature.properties.startTime);
    var triggerId = feature.properties.activityId;
    var msg = [];
    msg.push(feature.properties.name);
    msg.push('Start: ' + date.toLocaleDateString() + ' ' + date.toLocaleTimeString() + '');
    var dist = Math.round((feature.properties.totalMeters * 0.000621371)*100)/100;
    msg.push('Dist: ' + dist + ' mi');
    msg.push('Go to Activity');
    layer.bindPopup(msg.join(''), {maxWidth: 200});
}
```

The exciting bit happens in the onOpenActivity, and the RenderActivity functions when a user clicks the "Go to Activity" anchor in the popup. The onOpenActivity function instantiates an Activity Backbone model and invokes its fetch function to get the polyline for the selected activity ID. The renderActivity function passes in the success property of the AJAX call.

```javascript
onOpenActivity: function(event, popup) {
    var location = popup._latlng;
    this.map.closePopup(popup);
    // Capture the current center and zoom to restore map later
    this.originalCenter = this.map.getCenter();
    this.originalZoom = this.map.getZoom();
    this.activity = new Activity({activityId: event.target.id});
    var _this = this;
    this.activity.fetch({
        success: function () {
            _this.renderActivity();
        }
    });
}
```

When the polyline is successfully returned from the server, the renderActivity function creates a new activity map layer to swap with the marker cluster map layer. A feature GeoJson map layer is added to the map, along with the start and end markers.

```javascript
renderActivity: function() {
    $('.returnToSearch').show();
    if (this.map.hasLayer(this.activitiesLayer)) {
        this.map.removeLayer(this.activitiesLayer);
    }
    var props = this.activity.get('properties');
    this.map.fitBounds([
        [props.get('minLat'), props.get('minLon')],
        [props.get('maxLat'), props.get('maxLon')]
    ]);
    var style = {
        color: '#FF0000',
        weight: 3,
        opacity: 0.6
    };
    this.activityLayer = L.geoJson(this.activity.toJSON(),
        {style: style}).addTo(this.map);
    var polyline = this.activity.get('geometry').get('coordinates');
    var startPoint = polyline[0];
    var endPoint = polyline[polyline.length - 1];
    this.activityStart = L.marker([startPoint[1], startPoint[0]],
        {icon: this.startIcon}).addTo(this.map);
    this.activityEnd = L.marker([endPoint[1], endPoint[0]],
        {icon: this.endIcon}).addTo(this.map);
}
```
Once the user clicks Back to Search, this block of code restores the marker cluster layer. The activity feature layer, start, and end markers are removed (not shown), then the marker cluster layer is added back to the map, and the original center and zoom are restored.

```javascript
 this.map.addLayer(this.activitiesLayer);
 if (this.originalCenter != null && this.originalZoom != null) {
    this.map.setView(this.originalCenter, this.originalZoom, {animate: true});
    this.originalCenter = null;
    this.originalZoom = null;
 }
```

That's it! Be sure to go to the demo, http://www.exploringspatial.com/#demo/5, and click around.
