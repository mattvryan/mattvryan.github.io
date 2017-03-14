---
layout: post
title: Asynchronous Web Queries
published: True
---
I've come to the conclusion that the way we are handling queries in web applications is flawed.

It's a strong statement to make since what we have seems to have been working pretty well for us so far.  But I think mostly that's because the backend engineers have been propping an outdated model up for quite a while.  I think we will need to see a fundamental change in web application development as the next step forward.

Synchronicity is the problem here.  Synchronicity is an enemy of scale.  Synchronicity means that two systems that are working together on the same task must do so in harmony with each other, coordinating their efforts with each other.  When this is the case, the two systems must be treated as a single logical system, and not as two distinct systems, because they rely on each other.  Designers of backend systems intended to achieve high levels of scale will do whatever they can to eliminate synchronicity between services in order to allow each to scale independently and avoid situations where one system can impact another or where one system has tight dependencies on another.

All of this is mostly for naught, however, once the web client comes into play.  This is because the web client usually asks for a resource via a simple HTTP GET ... and then waits for a 200 OK accompanying the response to the request.

In other words, the client is dictating a synchronous request/response transaction.  On the back end, no matter what else is done or how else the system is designed or implemented, at some point a synchronous request must be handled in a synchronous way with a synchronous response.  So long as the client expects a synchronous response to requests, my attempts to eliminate synchronicity from the back end in order to achieve better scale are mostly irrelevant.  The architecture might support asynchronous processing, but I have to synchronize it at some point so it's basically synchronous anyway, despite my amazing asynchronous architecture.

What needs to happen instead is that web application designers need to start implementing asynchronous queries from the beginning.  The applications they develop need to be designed to be responsive and to adapt to changes that come in via the back channel.  Most developers of application frameworks for rich GUI apps on Windows, Mac, iOS, Android, etc. have this model built in - the framework requires you to create your app so you handle UI updates asynchronously.  So it can certainly be done for web frameworks too, and it is time we started insisting that web apps be built this way.

Here's a way it would work:
* A web application (client) issues an HTTP GET request.
 * The request may optionally be accompanied by a callback URL (e.g. a websocket).
* The server obtains the request, enqueues the request or stores it in a database for later processing or whatever, and then responds with 202 ACCEPTED.
 * If the request did not include a callback URL, the response would include a unique token for later use by the client, which token identifies this particular request/response pair.
 * The initial response DOES NOT contain any response data for the request - the response body is empty.
* Meanwhile the server has probably already asynchronously begun processing the response to the request.  Once done, the server sends the response to the callback URL if one was provided, or if not stores the response to wait for the client to ask for it.
* The client either waits for the callback URL to be invoked, or waits a period of time in a background thread and then polls the server using the token returned in the original response to identify the request/response pair.  If the response is now ready it is returned.

If web servers were to insist on this model, it would make for much more scalable web applications.