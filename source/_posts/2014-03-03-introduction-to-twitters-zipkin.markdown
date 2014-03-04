---
layout: post
title: "Introduction to Twitter's Zipkin"
date: 2014-03-03 15:35:33 -0500
comments: true
categories: [twitter, zipkin, open-source]
---

Zipkin is the Twitter open-source implementation of Google's distributed
tracing system, [Dapper]. It's a great tool for people who wants to understand
the bottleneck in their multi-services system. The only downside is that I
found its documentation isn't quiet clear about the tracing format, so I
decided to write a blog post that gives an overview of the system and its
communication protocol.

Before we continue, I would suggest you to take a glance at the paper. It would
give you some background knowledges and the assumptions of the Zipkin. I will
try to include relevant points in the post, but you may find it easier if you
read the paper first.

## Overview

Zipkin splits the roles of a tracing system into four parts: a collector, a
query service, a database and a web interface. Zipkin is a passive tracing
system, which means the app are responsible of sending the tracing information
to the Zipkin. Zipkin itself does not actively listen to the traffic on the
network, nor does it try to ping the application for statistics.

## Architecture

{% img /images/posts/zipkin-arch.png 650 "Zipkin Architecture" %}

An usual Zipkin deployment looks like the figure above. The recommended
database is Cassandra. The protocol between the applications and Zipkin
collector is Zipkin/Scribe/Thrift (read Zipkin on Scribe on Thrift). If you
want scalability, the zipkin project recommended to setup a full Scribe
environment. You can run multiple copies of Zipkin collector and configure your
server-local Scribe receiver to route Zipkin messages to the cluster for
load-balancing. For testing or low workload environment, you can point your
application directly to the Zipkin collector, as it supports Scribe protocol as
well.

## Tracing

Zipkin treats every user initiated request as a **trace**. Each trace contains
several **span**s, and each span is correlated to a RPC call. In each span, you
can have several annotations. There are four annotations the one span must have
in order to construct a full-view of a RPC call (in chronological order):
**cs**, **sr**, **ss**, **cr**, in which *c* stands for the client, *s* stands
for the server and the second *s* stands for send, the second *r* stands for
receive. Please note that these annotations does not have to be all present in
one span. You can send Zipkin two spans of the same spanID and have (cs, cr)
and (sr, ss) respectively, this is an useful property since you can do logging
in the client and the server separately. Each of those annotations would also
have a timestamp to denote when the event happened and an host for on which
host this event happened.

If you took a look at the Zipkin's thrift definition, you will also see that
the span also carries a list of binary annotations. These are a special kind of
annotations allows you to tag some request-specific information in the trace,
for example, the HTTP request URI, the SQL query or the HTTP response code.

### ID propagation

In the last section, we talked about trace and spans. Each trace is identified
by a globally unique traceId. Each span is identified by the traceId it belongs
to and an in-trace unique spanId. You may also specify an parentSpanId to
represent another RPC call made during the parent span's duration. The spans
should form an acyclic tree structure.

Now think about how server handles a request. Let's say we have a nginx server
as frontend, an application server and a database server. When nginx gets a
request, it needs to generate a traceId and two spans. The first spans denotes
the user requesting nginx, it will have **spanId = traceId** and **parentSpanId
= 0** by convention for root spans. The second spans will be generated when
nginx initiate the connection to the upstream. It would have a new spanId,
parentSpanId set to the first span's id and reuse the same traceId.

The nginx will then need to pass the second span's traceId and spanId to the
upstream. Fortunately, there's a convention for HTTP. Zipkin uses HTTP header
to carry those informations. The nginx would need to set `X-B3-TraceId`,
`X-B3-SpanId` and `X-B3-ParentSpanId` for the upstream to pick up, and the same
process goes on for each layer. If you're using other protocols, you might need
to come up with your own creative solution. For example, you may use SQL
comment to carry over those ids in database queries.

## In Practical

You should have enough knowledge of Zipkin to get started by now. Let's see how
these things would be use in code.

### Setup

Before we dig into codes, you need to deploy the Zipkin first. You can download
the Zipkin code and set it up yourself. To make things easier, I packaged
Zipkin into Docker images, enabling one-liner deployment. Check out
[docker-zipkin] if you're interested.

### Communications

We've talked about how Zipkin processes traces and spans. Here we will use an
example to show you what has been transferred to the collector under-the-hood.
We will reuse the example before: nginx frontend and an application server.
Note that you will not see any Trace object below, since trace is a virtual
entity that only exists as `traceId` in Spans. In the following example, we
will use JSON to denote an object since it's easier to write. Also, in the real
world Zipkin communication, spans are being encapsulated in Scribe. It would
look like this.

```json
{ category: "Zipkin", message: <Span-serialized-as-Binary-Thrift-Struct> }
```

When an user's request hits nginx, the nginx sends a span to the collector.

```json
{
  traceId: 1,      // randomly generated globally unique ID
  spanId:  1,      // root span shares spanId with traceId
  parentSpanId: 0, // root span does not have a parent
  name: "GET",     // RPC method name
  annotations: [
    {
      timestamp: "10", // a UNIX timestamp in **milliseconds**
      value: "sr",
      host: {
        ipv4: 0xC0A80101, // IP address, but as an Integer
        port: 80,
        service_name: "nginx"
      }
    }
  ],
  binaryAnnotations: [ // It's optional, useful for store metadata.
    {
      key: "http.uri",
      value: "/book/1990", // would be store as byte[]
      annotationType: "String",
      host: {
        ipv4: 0xC0A80101,
        port: 80,
        service_name: "nginx"
      }
    }
  ]
}
```

The nginx would than figure out that it needs to contact the upstream
application server to serve the content. Before it initiates a connection, it
sends another span to the collector.

```json
{
  traceId: 1,       // all spans in this request shares the same traceid
  spanId:  2,       // note that a new ID is being used
  parentSpanId: 1,  // the user <-> nginx span is now our parent
  name: "GET Book", // RPC method name
  annotations: [
    {
      timestamp: "12",
      value: "cs",
      host: {
        ipv4: 0xC0A80101,
        port: 80,
        service_name: "nginx"
      }
    }
  ],
  binaryAnnotations: []
}
```

The application server receives the request and, just like nginx, sends a
**server receive** span to the collector.

```json
{
  traceId: 1,
  spanId:  2,
  parentSpanId: 1,
  name: "GET Book",
  annotations: [
    {
      timestamp: "14",
      value: "sr",
      host: {
        ipv4: 0xC0A80102,
        port: 3000,
        service_name: "thin"
      }
    }
  ],
  binaryAnnotations: []
}
```

After the request has been processed, the application server sends **server
send** to the collector.

```json
{
  traceId: 1,
  spanId:  2,
  parentSpanId: 1,
  name: "GET Book",
  annotations: [
    {
      timestamp: "18",
      value: "ss",
      host: {
        ipv4: 0xC0A80102,
        port: 3000,
        service_name: "thin"
      }
    }
  ],
  binaryAnnotations: []
}
```

The nginx now receives the response from the upstream, it will sends a **cr**
to the collector. It also sends a **ss** before it proxies the response back to
the user.

```json
// client receive from upstream
{
  traceId: 1,
  spanId:  2,
  parentSpanId: 1,
  name: "GET Book",
  annotations: [
    {
      timestamp: "20",
      value: "cr",
      host: {
        ipv4: 0xC0A80101,
        port: 80,
        service_name: "nginx"
      }
    }
  ],
  binaryAnnotations: []
}

// server send to the user
{
  traceId: 1,
  spanId:  1,
  parentSpanId: 0,
  name: "/book/1990",
  annotations: [
    {
      timestamp: "21",
      value: "ss",
      host: {
        ipv4: 0xC0A80101,
        port: 80,
        service_name: "nginx"
      }
    }
  ],
  binaryAnnotations: [
    {
      key: "http.responseCode",
      value: "200",
      annotationType: "int16",
      host: {
        ipv4: 0xC0A80101,
        port: 80,
        service_name: "nginx"
      }
    }
  ]
}
```

### Send trace to Zipkin

#### Scala

Let's talk about Zipkin's native language: Scala first. Zipkin project
published a client library based on Scrooge and Finagle. To use the library,
you will need the following dependencies (shown in Gradle script format).

```groovy
repositories {
  mavenCentral()
  maven { url "http://maven.twttr.com/" }

dependencies {
  compile 'org.apache.thrift:libthrift:0.9.1'
  compile 'com.twitter:finagle-core_2.10:6.12.1'
  compile 'com.twitter:zipkin-scrooge:1.0.0'
  compile 'com.twitter:util-core_2.10:6.12.1'
  compile 'com.twitter:util-app_2.10:6.12.1'
}
```

For the code example, Twitter already have an great example on the github.
Please check out [zipkin-test].

#### Java

For Java, I would not recommend to use the Finagle Java support just yet. (or
maybe I'm too dumb to figure it out. :( ) Fortunately, there is a Zipkin
implementation in Java called, [Brave]. The dependencies you're looking for are
listed below.

```groovy
repositories {
  mavenCentral()
  maven { url "http://maven.twttr.com/" }

dependencies {
  compile 'com.github.kristofa:brave-impl:2.1.1'
  compile 'com.github.kristofa:brave-zipkin-spancollector:2.1.1'
}
```

Brave provides an awesome `ZipkinSpanCollector` class which automagically
handles queueing and threading for you.

## Conclusion

Phew, finally we can conclude this long blog post. These are basically where I
got lost when I tried to understand the Zipkin and tried to extend some other
services such as nginx, MySQL to report traces back to the Zipkin. I hope these
experiences would have you to get hands on the Zipkin faster. Zipkin actually
have more feautres than we talked about here, please also take a look at the
[doc] directory too. Have fun!

[Dapper]: http://research.google.com/pubs/pub36356.html
[docker-zipkin]: https://github.com/itszero/docker-zipkin
[zipkin-test]: https://github.com/twitter/zipkin/blob/master/zipkin-test/src/main/scala/com/twitter/zipkin/tracegen/Requests.scala
[Brave]: https://github.com/kristofa/brave
[doc]: https://github.com/twitter/zipkin/tree/master/doc
