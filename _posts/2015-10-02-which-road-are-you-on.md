---
layout: post
title: Which road are you on?
author: {{ site.author.name }}
---

## Background

My team at UAEU is designing a system that delivers changes in road speed limit to drivers in near-realtime as a senior project in Electrical Engineering. The system consists of a the driver's Android smartphone and a remote server. The backend has two components: 1) a Node.js server used to process requests, and 2) a MongoDB database containing the geographic data and speed limit for each road.

All communication between client and server is done over UDP to minimize data usage for the user. Each exchange (request + response) consumes around 150 bytes of mobile data on average. The format used is JSON, due mainly to widespread library support, and the fact that it doesn't add too much overhead.

I've been working on the backend for the past week. The server itself is relatively straightforward thanks to the Node.js standard library. However, I faced a few issues getting the main feature of our system to work properly: that is, determining whether or not the client is on a road that is stored in our database.

## First attempt

We initially thought that the whole problem could be solved using a simple bounds check on each coordinate pair, with each road represented as a series of coordinates. Our first MongoDB schema therefore looked something like this:

* Collection: *coordinates*
  - Field: *lat*
  - Field: *lng*
  - Field: *road_id*
* Collection: *roads*
  - Field: *name*
  - Field: *speed*

Suppose the server receives the request ```{"lat": 23.44, "lng": 51.55}``` from the client. A simple bounds check query for the above coordinate in MongoDB looks something like:

```javascript
{
  lat: {$lte: req.lat+0.00001, $gte: req.lat-0.00001},
  lng: {$lte: req.lng+0.00001, $gte: req.lng-0.00001}
}
```

If there is in fact a coordinate that lies within these bounds, we then run another query using the *road_id* stored with the document to find the road it lies on. Easy, right?

It turns out this approach doesn't work at all, for two main reasons.

Firstly, we are dealing with a projection of the Earth from 3D space to 2D space. As a consequence, geographic positions can't be compared with each other in such a linear manner.

Secondly, even if they could, our query *still* sucks since it doesn't take into account positions that lie purely horizontal, purely vertical, or a mix of both, relative to the target road coordinate.

## Third-party, please?

Fortunately for us, OpenStreetMap hosts a free reverse geolocation service called [Nominatim](http://nominatim.openstreetmap.org).

Given a coordinate pair, the API responds with information about the location extracted from OSM data in JSON or XML format. Google provides a similar service, but for our region the data from OSM is superior.

Request: [http://nominatim.openstreetmap.org/reverse?format=json&lon=55.656818&lat=24.195169](http://nominatim.openstreetmap.org/reverse?format=json&lon=55.656818&lat=24.195169)

Response:

```json
{
  "place_id": "76620021",
  "licence": "Data \u00a9 OpenStreetMap contributors, ODbL 1.0. http:\/\/www.openstreetmap.org\/copyright",
  "osm_type": "way",
  "osm_id": "103242220",
  "lat": "24.1998026",
  "lon": "55.6679748",
  "display_name": "Khalifa Bin Zayed Street, Asharej, Abu Dhabi, United Arab Emirates",
  "address": {
    "road": "Khalifa Bin Zayed Street",
    "village": "Asharej",
    "state": "Abu Dhabi",
    "country": "United Arab Emirates",
    "country_code": "ae"
  }
}
```

Thanks to the unique `osm_id` returned with the response, we can identify each road in the database, allowing us to remove the *coordinates* collection entirely.

But the API is not without its disadvantages. Nominatim API is free, but only for light usage: they advise no more than 1 request per second. In addition, even for simple testing, the free API is horribly slow. It takes around 800 ms on average to get a response from their servers.

Better (expensive!) usage tiers are provided by other companies, such as Mapbox, but the reliability of our system clearly decreases if we rely on any external service, as it introduces an additional point of failure outside of our control. This also limits future scalability since the cost of maintaining the system will increase drastically with the number of users.

The code that uses this approach can be found [here](https://github.com/aksiksi/gp2-server-node/blob/master/server.js).

## A proper solution

I went back to the drawing board, and started searching for techniques in GIS (Geographic Information Systems) that allow us to check whether or not a point is on a road. It turns out that this is a trivial thing to do, from a GIS point of view.

I started by exporting all geographic data for my city, Al Ain, using the OSM online export tool. Next, I imported all roads located in the data file into QGIS as a separate layer; [here](http://learnosm.org/en/osm-data/osm-in-qgis/) is an excellent guide outlining how to do that.

OSM already provides road data in *LineString* format i.e. a sequence of coordinates. However, that is not enough to get the job done in our case. So we need to use what is known in the GIS world as a *Buffer*.

Given a point or series of points, a *Buffer* creates a *Polygon* that extends outward from these points, with some given radius. Let me show you an example.

Here is a sample road segment (highlighted in yellow).

![Road segment selected on OSM map]({{ site.url }}images/post-1/1.png)

And here is the result of applying a *Buffer* to it with a radius of 4 meters.

![Result of applying buffer with 4m radius]({{ site.url }}images/post-1/2.png)

Great, so now we have a *Polygon* that represents this road segment. The question now is, how can we use this with MongoDB?

One of the standard shape (or *feature*) representation formats used in GIS is called GeoJSON. Luckily for us, MongoDB can perform geospatial queries on shapes in this format, and QGIS has built-in support for exporting shapes to GeoJSON.

To accommodate for this change, I added a *segments* collection containing each segment of a road with a different speed limit. Each segment document has a `shape` field that contains the *Polygon* exported from QGIS in GeoJSON format.

To find which road segment, if any, a given coordinate lies on, we can use a simple query in Node.js:

```javascript
var segments = db.collection(SEGMENTS);

// GeoJSON coordinate representation
const loc = {
  type: 'Point',
  coordinates: [parsed.lng, parsed.lat]
};

// Perform query on collection
// Find the road segment that contains Point `loc`
const q = {
  shape: {
    $geoIntersects: {
      $geometry: loc
    }
  }
};

segments.findOne(q, {speed: 1, road_id: 1, _id: 0}, (err, segment) => {
  // Do whatever here!
});
```

With this approach, the response time of the server is decreased by a factor of 10 on average - from 800 ms to around 70 ms. The disadvantage of course is that roads have to be selected, buffered, exported, and added to the database by hand.

This process can be automated by parsing the exported OSM data file manually and using a GIS processing library that can do the buffering for us, such as [Turf.js](https://www.mapbox.com/guides/intro-to-turf/) from Mapbox.

However, for our senior project, I see no need to go through the extra effort since we only plan on testing the system with 2 or 3 roads.

You can follow the progress of our project, **roadomatic**, at its repo on [Github](https://github.com/aksiksi/roadomatic).
