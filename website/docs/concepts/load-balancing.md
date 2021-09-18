---
id: load-balancing
title: Load Balancing
sidebar_label: Load Balancing
---

Load Balancing means efficiently distributing the network traffic across multiple machines to balance out the load and prevent any hotspots. Load Balancers also keep track of servers which are not functional and avoid sending requests to those machines. When the server comes back up or new servers are added, Load Balancer will resume the distribution of traffic to that server again.

Apart from routing, Load balancers might also do other activities like **SSL termination**, which means SSL traffic is decrypted by the load balancer and then passed on to the server. This saves the work of SSL decryption on the web servers improving performance and reducing latency.

Load balancing can happen at Layer 4 (L4- Transport) or Layer 7 (L7- Application) of the OSI Model. Also, there are different kinds of Load Balancers.

## Hardware Load Balancer or Software Load Balancer

**Hardware-based load balancers** are high-performance solutions where specialized processors are used to achieving optimum performance. To handle increased loads you need to buy more machines. Hence they are generally a costly solution and require specialized maintenance. Due to the above limitations, these are not very popular. The most widely used ones are Citrix Netscaler and F5.

**Software-based load balancers** uses generic hardware to run and are easy to scale and flexible in configuration. Some of the popular ones come as installable software and others are provided as a Service called Load Balancer as a Service (LBaas). Popular software-based load balancers included Nginx, Varnish, HAProxy, and LVS. Popular LBaas include AWS ELB and Stratoscale LBaas. LBaas are managed by the cloud vendors and handle fault tolerance and elasticity for the users.

Check out the following [video of how Facebook handles billions of request using their unique load balancing techniques.](https://www.youtube.com/watch?v=bxhYNfFeVF4)

## Load Balancing Layer

Load Balancing can be introduced at any layer in the application increase scalability depending upon the number of requests served.

![Load Balancing](/img/lb.png)

- **Frontend Layer** - The most common use case of the load balancers is at the frontend of the application, between the client and the webserver. This helps to increase the number of requests that can be served by the system. Also, SSL Termination is done here to save CPU cycles of the application server.

- **Application Layer** - Load Balancers are placed between the webserver which is taking the requests and the application servers which are doing CPU intensive tasks. This helps in the proper utilization of the application servers without overloading them.
- **Persistance Layer** - Load Balancers are placed between the Application Layer and the Persistence Layer to serve more data requests without overloading Database servers.

## Load Balancing Algorithms

Several popular load balancing algorithms are used by Software-based load balancers. Depending upon the type of system and workloads, a particular scheme may be well suited than others.

1. Round Robin: In this scheme, the load balancer forwards requests to different nodes sequentially in a round-robin fashion, This is simpler to implement, however, it is not very efficient as it doesn't account for the current load on the server, which may already be loaded when a new request is assigned. Hence is mostly suited for simpler use cases where loads are not processor intensive.

2. Least Connection: The requests will be forwarded to a server that has the least number of connections or serving the least number of requests at that point in time. This delivers better performance than the round-robin, but again suited not suited for workloads where the single request might load the server.

3. Least Response Time: This scheme is based upon the time taken by the server to respond to a dummy request sent to a server selected by an algorithm based on the least connections and response time.

4. Hash-Based: Requests are distributed based upon a defined key example: Request URL, Request IP address.

5. Custom Load Algorithm: In this scheme, load balancer determines which server to forward the request to based on a combination of server metrics like Memory usage, CPU Usage, Response Time.

## Summary

Load balancer has become an essential component of any large scale system as it helps to balance loads across multiple machines. As the systems become more complex, increasingly popular and traffic volume surges, Load balancers act as a traffic cop to route the loads systematically, preventing uneven loads and performance issues on the systems. We discussed different types of Load Balancers and algorithms, in a system design interview based upon the problem at hand a particular configuration may be well suited than the other.
