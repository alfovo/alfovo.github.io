---
layout: post
title: WebSockets & Protocol Suite Etiquette
date: 2020-05-22 14:49:03 -0500
tags: [research, etiquette]
thumbnail: /assets/img/design/etiquette.svg
---

I owe my livelihood to the vast network of servers chatting with each other online. I'm heartened by web applications that send data so quickly and reliably that their physical distance is imperceptible to me, the user. When you factor in the value and variability of information passed around online, it all seems so risky and precarious. So how _do_ servers conduct themselves when sending data to each other across the wild, wild web? What is the proper protocol for sending and receiving virus-free content?

Most of the web is based on the _HyperText Transfer Protocol_ (HTTP) request/response model of client-initiated transactions. Your browser, the client, politely requests data from a server and the server responds to those requests. Even better, when following the _Transmission Control Protocol_ (TCP) on which HTTP is based, the client will send an acknowledgement that the data has been received so the server can resend if the data gets lost. Because of this behavior, TCP is described as a connection based, stream-oriented protocol.

Consider the _User Datagram Protocol_ (UDP), an alternative to TCP. While the communication is still initiated by the client, this protocol is "connectionless" or message-oriented, because the server will send data without checking on the state or readiness of the client. This is the late 90s coffeeshop slam poetry of data transfer: the server will refuse to do normal audience banter, instead shouting a jumbled bunch of datagrams and then dropping the mic.

By boldly forgoing the customary handshake, the hassle of worrying if the data has been received by the client, and any guarantee that the data will be sent in order[^1]; UDP offers low latency data transfer for bandwidth-intensive applications that can lackadaisically tolerate packet loss and disarray, like multiplayer games or, everyone's favorite new hobby, video conferencing.

Alas, wouldn't it be cool if we had UDP support in the browser?[^2] I can only naively imagine how much better audio/video streaming would be.[^3]

## And Now, Double Time

<div class="image-series">
  <img class="prototype" src="/assets/img/design/etiquette2.svg" alt="etiquette book by Emily post"/>
</div>
> **The test of good manners is to be able to put up pleasantly with bad ones.
-- Wendell Willkie**

Like etiquette, data transfer protocols are complicated and evolving. As a connection-based protocol, HTTP has been called too "chatty", because it requires so much reassuring back and forth between client and server. In contrast, UDP could be called too "venty" because it does not give a hoot about what the client has to say while communicating a torrent of information.

As web applications with high-frequency request-response cycles (real-time web applications if you will[^4]) became more popular in the mid-2000s, the traditional HTTP model no longer seemed appropriate. Setting up a new connection every request/response cycle introduced too much latency. Ideally there would be a persistent connection between client and server that was bidirectional, allowing the server to send data to the client without the client having to request it. The umbrella term "Comet", coined in a blog post[^5] by Alex Russell in 2006, was used for techniques approximating real-time back-and-forth communication between a web page and server.

The most popular solution was long polling, where a XMLHttpRequest connection to the server was held open until ongoing communication was no longer required. However, long polling was not ideal because the cookie and header data within the HTTP request made data transfer bandwidth intensive, and multiple simultaneous HTTP requests from the same client could scramble message order.

Was it possible to create a bidirectional connection based protocol with the reliability of TCP, that was low latency and message-oriented like UDP, and also built to address all the concerns that arise when operating in a web environment?

## Enter, WebSockets

<div class="image-series">
  <img class="prototype" src="/assets/img/design/internet_protocol.svg" alt="Internet protocol intrigue"/>
</div>
> **“The world was my oyster but I used the wrong fork.”
-- Oscar Wilde**

The frustration and limitations of Comet inspired developers Michael Carter and Ian Hickson to introduce a new standard for modern real-time, bi-directional communication on the web that they coined ‘WebSocket’[^6]. In 2010 Google Chrome 4 was the first browser to fully support WebSockets and today all major browsers support WebSockets.

WebSockets provide a persistent, low latency connection that can support transactions initiated by either the client or server[^7]. To establish a WebSocket connection, the client uses standard HTTP protocol to request an "upgrade" from HTTP to a WebSocket connection. If the server supports WebSockets it will respond with an _HTTP/1.1 101 Switching Protocols_ response, which replaces the initial HTTP connection with a WebSocket connection that uses the same underlying TCP/IP connection.

In other words, WebSockets servers authenticate clients with that classic TCP handshake over HTTP and then, after upgrading to a WebSocket connection, repurpose that underlying TCP connection. The intent is to provide the thinnest possible transport layer on top of TCP while handling some web-specific concerns.

## Enter, Moi

WebSockets are great for the chat application I work with at my current job because:

1. They don't require the client to poll the server for new messages, instead the server can send messages to the client whenever data becomes available.
2. They use a frame-based messaging system that reduces the amount of non-payload data that is transferred, reducing latency.
3. They provide a mechanism to detect dropped (disconnected) clients and can handle up to 1024 connections per browser.

A downside to using WebSockets is that they don’t automatically recover when connections are terminated. Luckily, the client-side library we use does this for us.

I enjoyed learning about WebSockets for work and with the confidence of a beginner I wonder, would it be possible to build WebSockets on a UDP rather than TCP connection? Would this make them feel closer to the transport layer protocol the word "socket" implies, rather than a tag along to HTTP? Surprise me!

WebSockets had a contentious beginning[^8] and a very unstable spec for some time[^9]. It reminds one of Princess Maria Argyropoulina, who scandalized Venice by having guests at her wedding use the novel fork in place of their God-given hands[^10]. St. Peter Damian, a hermit and ascetic, was so offended by the princess's daintiness that he declared her untimely death from the plague a just punishment by God for her vanity. Maybe at first WebSockets seemed like a vestigial limb growing out of HTTP, but like forks, we can't imagine living without them now.

#### Footnotes

[^1]: Doing away with these social niceties allows the UDP header to be [only 8 bytes compared to TCP's 20 bytes](https://www.geeksforgeeks.org/user-datagram-protocol-udp/).
[^2]: I wonder if there was talk about adding UDP support for the web during the browser wars raged between Netscape and Microsoft from 1995 to 2001, but I wasn't there, I was spilling my guts to SmarterChild at my best friend's house.
[^3]: While reading gossipy comparisons of TCP to UDP, it struck me that sacrificing reliability comes at an exceptionally high cost in computing. Content from TCP "heads" reminds me of the steadfast popularity of ACID-compliant data stores, like PostgreSQL, despite their age and decided unfashionableness. By the way, top TCP head content is "I could tell you a joke about UDP, but you might not get it."
[^4]: The term 'real-time' always makes me laugh. A 'real-time' application sounds like you are applying to become a fourth-dimensional being.
[^5]: <https://infrequently.org/2006/03/comet-low-latency-data-for-the-browser/>
[^6]: You can still find the [IRC chat](https://krijnhoetmer.nl/irc-logs/whatwg/20080618#l-1145) and the [W3C mailing list](https://lists.w3.org/Archives/Public/public-whatwg-archive/2008Jun/0165.html) where they started planning out WebSockets. Personally I think the coolest part is when they talk about whether not wanting your IRC chat logged is a valid privacy concern. Someone says it's antisocial, another (jokingly?) that it's so not web 2.0. Prescient.
[^7]: There is a way, standardized as part of HTML5, to send push notifications from server to client with regular old HTTP: [Server-sent events](https://en.wikipedia.org/wiki/Server-sent_events). The WebSocket use case is _bidirectional_ communication.
[^8]: <https://www.cnet.com/news/web-sockets-and-the-risks-of-unfinished-standards/>
[^9]: <https://docs.microsoft.com/en-us/previous-versions/msdn10/hh563510(v=msdn.10)>
[^10]: You can read a great article on the history for forks [here](http://www.slate.com/articles/arts/design/2012/06/the_history_of_the_fork_when_we_started_using_forks_and_how_their_design_changed_over_time_.html)
