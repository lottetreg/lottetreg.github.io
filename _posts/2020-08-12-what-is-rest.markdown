---
layout: post
title:  "What is REST (really)?"
date:   2020-08-12
---

Michael Stowe's book, [_Undisturbed REST: A Guide to Designing the Perfect API_](https://www.mulesoft.com/lp/ebook/api/restbook), begins with an understanding of what REST is. REST, or Representational State Transfer, is an architectural pattern for APIs, and it was defined by Dr. Roy Fielding in his [2000 Doctorate Dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm).

REST is extremely flexible when compared with other API architecture patterns, like SOAP and RPC—it can return any content-type, and it can be used with any protocol (although it is typically used with HTTP).

A RESTful API must obey the following constraints:

1) Client-Server<br>
2) Stateless<br>
3) Cache<br>
4) Uniform Interface<br>
5) Layered System<br>
6) Code on Demand (optional)

## Client-Server

The client and the server are separate, allowing for each to grow individually without impacting the other.

## Stateless

The data within a request is all that is needed for that request to successfully complete. It does not rely on any data stored on the server or within a session.

## Cache

The caching is handled by the client, but the server's response tells the client whether the data should be cached (and for how long), or whether it should be received in real-time, and therefore not cached.

## Uniform Interface

The interface between the client and the server does not change, and it is decoupled from both applications' backends. (There are additional constraints that make for a truly uniform interface, but I'm leaving those out for simplicity's sake.)

## Layered System

The API is made up of layers that increase its flexibility, scalability, and security—for example, you may use an API Management Proxy to handle load balancing and routing.

Using layers helps keep your API modular, so you can easily swap out systems and keep your modules loosely coupled. It can also improve security—you may be able to stop attacks at the proxy layer, or within other layers, before they reach the actual server architecture. This proxy layer creates a single point of access, preventing clients from interacting directly with potentially vulnerable parts of your system.

As a note on security in general—good security relies on multiple layers of protection, with the understanding that some of them may be bypassed. There is no single, "stop-all" solution to preventing attacks.

## Code on Demand

Code on Demand allows for code to be transmitted via the API, creating smart applications that are no longer solely dependent upon their own architecture. Code on Demand has not been widely adopted, most likely because Web APIs are consumed across multiple languages, and the transmission of code raises security questions and concerns.

Code on Demand is the only optional REST constraint.
