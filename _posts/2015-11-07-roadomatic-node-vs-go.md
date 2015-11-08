---
layout: post
title: "Roadomatic: Node vs. Go"
author: {{ site.author.name }}
---

<div class="message">
<b>Reader beware:</b> this is a code-heavy post.
</div>

## Contents

<div class="contents">
  <ul>
    <li><a href="#overview">Overview</a></li>
    <li><a href="#server-operation">Server Operation</a></li>
    <li><a href="#round-1-request-processing">Round 1: Request Processing</a>
      <ul style="margin: 0;">
        <li><a href="#udp-socket">UDP Socket</li>
        <li><a href="#request-validation">Request Validation</a></li>
      </ul>
    </li>
    <li><a href="#round-2-querying-the-database">Round 2: Querying the Database</a></li>
    <li><a href="#round-3-performance">Round 3: Performance</a></li>
    <li><a href="#conclusion">Conclusion</a></li>
  </ul>
</div>

## Overview

As I've outlined in my [previous]({% post_url 2015-10-02-which-road-are-you-on %}) post, we are building a system called [Roadomatic](https://github.com/aksiksi/roadomatic) for our senior project in Electrical Engineering at UAE University. The goal of the system is to enable delivery of location-based road and traffic information to drivers in real-time.

Roadomatic consists of two main components: a client application targeting the Android operating system, and a server written in Node implemented by yours truly.

From the very start, our number one priority has been efficiency. For example, the system's communication protocol consists of JSON on top of UDP, allowing for easy encoding and decoding on both ends, as well as data usage on the order of 200 bytes/exchange.

In this post, I'll be talking mainly about the backend, and specifically comparing two functionally identical implementations of the server I've written in Node and Go.

## Server Operation

The backend consists of two main parts:

* A UDP server
* A MongoDB database for data storage

The general flow of the server operation is as follows:

1. A UDP datagram is received on the socket.
2. Datagram contents are decoded using JSON, and geographic coordinates are stored.
3. The database is queried to find the road the received coordinate is on.
4. The response object is built, then encoded using JSON.
5. A datagram containing the response bytes is sent back over the wire.

As you can see, server-side processing is minimal in the system's current state. However, we might opt to defer more processing to the server in future changes.

But how is this processing done in each implementation? And how difficult were the two implementations relative to each other?

## Round 1: Request Processing

One of the reasons behind why we initially chose to transmit data as JSON was the technology stack we used to build the backend: Node and MongoDB. Both of these rely heavily on JSON (or BSON for the pedants). So as expected, working with data and querying the database were very easy to achieve in Node.

However, achieving the same in Go wasn't that much more complicated. I was pleasantly surprised by how easy many "complex" operations were in Go, given that it's a comparatively low-level language.

As for the UDP server itself, both languages have standard libraries that simplify the process immensely.

### UDP Socket

To start off, we'll compare between the two languages when it comes to listening on a port and executing code when a datagram is received.

```javascript
// Socket setup
var socket = dgram.createSocket('udp4');
socket.bind(config.server_port, config.server_host);

socket.on('listening', () => {
  console.log('Listening on %s:%d...', config.server_host, config.server_port);
});

socket.on('message', (req, remote) => {
  console.log('Message #%d received!', (++c));

  processRequest(req, remote, socket);
});

socket.on('error', (error) => {
  console.log(error.stack);
  socket.close();
});

// Request handler
var processRequest = (req, remote, socket) => {
  // ...
}
```

Let's see how it's done in Go:

```go
func main() {
  // Socket setup
  addr := net.UDPAddr{
    Port: PORT,
    IP: net.ParseIP(HOST),
  }

  conn, err := net.ListenUDP("udp", &addr)
  if (err != nil) {
    panic(err)
  }
  defer conn.Close()

  for {
    // Create a buffer for each request
    buf := make([]byte, 1024)

    // Read bytes into buffer (blocking)
    b, addr, err := conn.ReadFromUDP(buf)
    if (err != nil) {
      panic(err)
    }

    // Spawn a goroutine for each request
    go udpHandler(buf, b, n, conn, addr)
  }
}

func udpHandler(buf []byte, b int, ...) {
  // ...
}
```

### Request Validation

Next, let's look at how checking the validity of a request is implemented.

In both cases we're continuing from the end of the two snippets shown above.

For Node, below is the code responsible for checking whether or not the received request `Buffer` is valid:

```javascript
var processRequest = (...) => {
  try {
    parsed = JSON.parse(req);

    // Check for correct params
    if (!parsed.lat || !parsed.lng) {
      throw Error();
    }
  } catch (e) {
    console.log(e.stack);
    valid = false;
  }

  if (valid) {
    // Continue processing
  }
}
```

In Go, we're checking a `[]byte` instead of a `Buffer`, but the idea is the same:

```go
type Request struct {
  Lat float64 `json:"lat"`
  Lng float64 `json:"lng"`
}

func udpHandler(...) {
  // Slice buffer depending on read bytes, trim spaces
  clean := bytes.TrimSpace(buf[:b])

  // Parse received JSON
  r := Request{}

  err := json.Unmarshal(clean, &r)
  if err != nil {
    fmt.Println(err)
    return
  }

  // Continue processing
}
```

### Result: a tie

The line count was similar, and I think both versions are clear even for someone who's not too familiar with either Node or Go.

## Round 2: Querying the Database

This is the part where we expect Node to shine given its native support for JSON and asynchronous programming style.

First, a quick overview of what the code needs to do. We need to make two `find` queries to the database.

The first is to determine the road segment the given coordinate lies on using MongoDB's geospatial functionality and [GeoJSON](http://geojson.org/).

The second query is to extract further information  about the road in general, specifically the name of the street, using the `road_id` field stored with each `segment`.

The Node implementation uses the [mongodb](https://www.npmjs.com/package/mongodb) package, whilst the Go version depends on [mgo](http://labix.org/mgo).

```javascript
var findRoad = (parsed, db, callback) => {
  const resp = newResponse();

  // GeoJSON coordinate representation
  const loc = {
    type: 'Point',
    coordinates: [parsed.lng, parsed.lat]
  };

  var segments = db.collection(config.segments);

  // Perform query on SEGMENTS collection
  // Find the road segment that contains Point `loc`
  const q = {
    shape: {
      $geoIntersects: {
        $geometry: loc
      }
    }
  };

  segments.findOne(..., (err, segment) => {
    if (err) {
      // Collection read error
      console.log(err);
      resp.online = 0;
      callback(resp);
    } else if (!segment) {
      // No segment match!
      callback(resp);
    } else {
      // Segment match!
      var roads = db.collection(config.roads);

      // Find road name using road_id
      roads.findOne(..., (err, road) => {
        if (err || !road) {
          // Read error, return what we have
          console.log(err);
          resp.name = "";
        } else {
          resp.found = 1;
          resp.speed = segment.speed;
          resp.name = road.name;
        }

        callback(resp);
      });
    }
  });
};
```

Well... I don't really think async makes things clean.

Let's look at how we did the same in Go.

```go
func findRoad(req *Request) Response  {
  r := Response{Online: 1}

  lat, lng := req.Lat, req.Lng
  loc := bson.M{
    "type": "Point",
    "coordinates": []float64{lng, lat},
  }

  // Query database, retrieve road_id
  session, err := mgo.Dial(fmt.Sprintf("mongodb://%v:%d", MONGO_HOST, MONGO_PORT))
  if err != nil {
    r.Online = 0
    fmt.Println(err)
    return r
  }
  defer session.Close()

  segments := session.DB(MONGO_DBNAME).C(MONGO_SEGMENTS)
  s := Segment{}

  filter := bson.M{"_id": 0, "speed": 1, "road_id": 1}
  query := bson.M{
    "shape": bson.M{
      "$geoIntersects": bson.M{
        "$geometry": loc,
      },
    },
  }

  err = segments.Find(query).Select(filter).One(&s)
  if err != nil {
    fmt.Println(err)
    return r
  }

  roads := session.DB(MONGO_DBNAME).C(MONGO_ROADS)
  road := Road{}

  err = roads.Find(bson.M{"_id": s.RoadId}).One(&road)
  if err != nil {
    r.Online = 0
    fmt.Println(err)
    return r
  }

  r.Found = 1
  r.Speed = s.Speed
  r.Road = road.Name

  return r
}
```

### Result: Go wins

I'm no Go expert, but I believe such examples demonstrate how Go's approach to error checking really shines. For me at least, the Go version looks **much** cleaner.

Of course, if you think the Node version could have been written in a more elegant manner, please don't hesitate to leave a comment below. Or better yet, send us a PR on [Github](https://github.com/aksiksi/roadomatic)!

## Round 3: Performance

Here is a table outlining the results of testing. Please keep in mind that response time includes RTT from client (located [here](https://www.google.ae/maps/place/United+Arab+Emirates/@24.2947031,49.4553835,6z/)) to server (located [here](https://www.google.com/maps/place/Netherlands/@52.1917355,3.0369517,7z/)).

<table>
  <tr>
    <th></th>
    <th>Memory Usage</th>
    <th>Response Time</th>
    <th>Compiled</th>
    <th>Size</th>
    <th>Dependencies</th>
  </tr>
  <tbody>
    <tr>
      <td><b>Node</b></td>
      <td>25 MB</td>
      <td>150 ms</td>
      <td>No</td>
      <td>N/A</td>
      <td><a href="https://www.npmjs.com/package/mongodb">mongodb</a></td>
    </tr>
    <tr>
      <td><b>Go</b></td>
      <td>1.5 MB</td>
      <td>150 ms</td>
      <td>Yes</td>
      <td>6 MB</td>
      <td>None</td>
    </tr>
  </tbody>
</table>

### Result: Go wins

Go clearly is the winner here, by a large margin.

The memory usage of the Go implementation is simply astounding, allowing the backend to run on a constrained server. In our case, we're running the Node version on a 512 MB box, and memory usage is almost at the edge. Given MongoDB's memory requirements, it's good to shave off some MBs when given the chance.

## Conclusion

In summary, both Node and Go are excellent choices for writing server-based applications. However, for our specific use-case, Go is simply superior in almost every way.

We are currently still using the Node implementation for testing, but we will likely shift to the Go version if we end up deploying [Roadomatic](https://github.com/aksiksi/roadomatic) in production.

In any case, porting the system from Node to Go was an excellent experience for me personally. I've always wanted to learn Go, and this was the best kind of excuse to just do it.

<script>
  (function() {
    anchors.add('h2, h3, h4');
  })();  
</script>
