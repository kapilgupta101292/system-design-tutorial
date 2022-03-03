---
id: websockets
title: WebSockets
sidebar_label: WebSockets
---

Websocket is essentially a transport layer built on the top of the TCP/IP Protocol. and provides a persistent bidirectional communication channel between a client and a server.
The initial connection is established similar to a normal HTTP request and then upgraded to a lightweight and real-time bidirectional channel which allows the server to send downstream messages to the connected clients.

![Websocket Connection](/img/websocket/websocket.jpeg)

Websockets allow exchanging messages using any protocol as long as both client and server agree on the same for example JSON, XML, and any more.

Before the WebSockets, achieving the same functionalities would require constant polling the server which is generally high latency and overloads the backend servers.

Starting from around 2010, WebSockets are supported on all platforms including web and mobile devices. The standard for the WebSocket Protocol (RFC 6455 – The WebSocket Protocol) was published by IETF in 2011.
Backend servers can go for any client authentication mechanism for example cookie-based, HTTP, or TLS authentication.

## Challenges with using Websockets (WIP)
## Websocket Implementations(WIP)
