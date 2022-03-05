---
id: websockets
title: WebSockets
sidebar_label: WebSockets
---

WebSockets is essentially a transport layer built on the top of the TCP/IP Protocol. and provides a persistent bidirectional communication channel between a client and a server.
The initial connection is established similar to a normal HTTP request and then upgraded to a lightweight and real-time bidirectional channel which allows the server to send downstream messages to the connected clients.

![WebSockets Connection](/img/websocket/websocket.jpeg)

WebSockets allow exchanging messages using any protocol as long as both client and server agree on the same for example JSON, XML, and any more.

Before the WebSockets, achieving the same functionalities would require constant polling the server which is generally high latency and overloads the backend servers.

Starting from around 2010, WebSockets are supported on all platforms including web and mobile devices. The standard for the WebSocket Protocol (RFC 6455 – The WebSocket Protocol) was published by IETF in 2011.
Backend servers can go for any client authentication mechanism for example cookie-based, HTTP, or TLS authentication.

## WebSocets Usecases

Websockets are used in a myraid of applications which demands low latency, realtime connection for exchanging data between client and server.

- Notification/ Chat messages
- Multiplayer online games
- Live Sports commentary / ticker
- Realtime Social Network updates.
- Realtime monitoring

## Considerations while using WebSockets

- Security Issues: WebSockets are prone to common security vulnerabilities linked to the HTTP Protocol such as DDos, Authentication and Authorization issues, Sniffing, Cross-Site Websocket Hijacking (Similar to CSRF (Cross-Site Request Forgery)) 
- Connection Issues : WebSocket connection, once terminated don't recover automatically and need to be handled. However, Reconnection logic is generally handled by the available client side libraries.

## WebSockets Implementations
There are a plenty of popular WebSocket implemenations/libs available in the most popular languages, which makes it easy to get up and running easily.

- Javascript : WS, Sockets.io, SockJS 
- Java : javax.websocket-api, java.net.Socket, Jetty
- Ruby : EventMachine, websocket-client-simple, em-websocket
- Python : pywebsocket, Tornado


