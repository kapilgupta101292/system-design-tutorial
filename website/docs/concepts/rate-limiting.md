---
id: rate-limiting
title: Rate Limiting
sidebar_label: Rate Limiting
---

Rate Limiting means to throttle the number requests to your service from a particular source (user, device, IP, location, etc) to some maximum limit. The requests submitted over the limit are either immediately rejected or they are delayed. Rate limiting allows us to create resilient services, that can handle various scenarios discussed below.

![Working of Rate Limiter](/img/rate-limiter/rate-limiter-working.png)

The simplest example of rate limiting is that you are allowed to enter 3 incorrect passwords before your bank account is locked for online transactions for a day. Similarly, the Github API allows around 5000 requests from a user account per hour, after that, you will get an error to wait for some time before sending another request.
Services typically send 503 (service unavailable) or 429(Too many requests) Http status code when the limit is exceeded.

## Advantages of Rate Limiting

- Security: Rate Limiting is implemented to prevent DDoS attacks which overwhelms the servers which are a costly affair for any company both in terms of money and customer experience. Rate limiting your APIs will also limit the amount of data that gets exposed when security is somehow compromised.

- Protection against faulty Clients: Rate Limiting is also useful to make your system foolproof against any faulty or malicious client software, which might be sending a lot of requests due to a software bug.

- Cost Optimization: Rate Limiting Computationally intensive API will help optimize the cost of infrastructure.

- Maintain Quality: If you have a public API, implementing rate limit offers a better experience for all the users by distributing the number of requests that can be served. It prevents the Noisy Neighbour problem when one user utilizes too much-shared resources, such that it causes higher latency or higher failure rates.

- Maintain Priority: Several API's have paid version and free version, by assigning lower rate limits to the free users, you can make sure that Paid users get a better experience for their money.

### Types of Rate Limiting

Due to ever-increasing loads on the servers for popular services, almost all companies use rate-limiting in their services, If you are also creating such services, it is very important that you add some form of rate limiting to prevent your infrastructure and customer base.

We can implement rate limiting based upon various methods and parameters that can be defined when setting rate limits. Based on the security and business requirements, we can choose one of the following criteria.

- **User rate limiting**: This is the most popular criteria used for rate limiting. Based on the number of requests from a particular User's IP or API Key, requests will be throttled once the limit is reached. Users will be shown the appropriate status code to reflect that their requests are throttled.

- **Geographic rate limiting**: Based upon the security or business requirements, We can set rate limits based on the geographic regions, This could reduce the likelihood of DDOS attacks and other suspicious activity.

- **Resource Based Rate limiting**: We can also employ flexible Rate Limits based upon the amount of resources available like CPU, network bandwidth, etc. When resources are constrained, we can reduce the rate limits, Once the resources are available, we can increase the rate limits.

- **Hybrid rate limiting**: We can combine certain characteristics of the above rate limiting algorithm to achieve optimum results, like a user can send 100 requests per second from a particular IP address in a geographic region.

## Requirement Gathering.

### Function Requirements

- For a given request, return a boolean value of whether the request is throttled or not.

### NonFunction Requirements

- **Low Latency** - We want the rate limiting module to be as fast as possible, otherwise it adds latency to the processing time of the request.

- **Accurate** - Our rate limiting module should be as accurate as possible. It should not be throttling requests which should have actually gone through.

- **Scalable** - Our service should be highly scalable to support more number of requests.

In case the rate limiting service is not available, the system should not block the processing of the request.

### Rate Limiting Algorithms

- **Token Bucket Algorithm** - In this algorithm, we place n tokens in a bucket, every bucket has a maximum capacity and is filled at a constant rate, ex 10 tokens pers second. When a request for our API is received, tokens are withdrawn from the bucket if available. If required number of tokens are not available then the request is rejected or delayed. Eventually, the bucket is refilled with tokens and the client can make more requests to our API. Please note that depending upon the operations in the API request, one request might need more than one token.

* **Leaky Bucket Algorithm** - It is one of the simplest rate limiting algorithms, It implements rate limiting using a bucket concept per user (depending upon the factor on which rate limiting is applied) which holds all the incoming requests of that user. The bucket is of the fixed size corresponding to the number of requests we want to allow. When a request is received, it is put into the bucket. Requests are picked up from the other end of the bucket (more like a queue than a bucket) for processing. If the number of requests at any point in time exceeds the size of the bucket, those will leak and will not be considered. It can lead to queueing up of old requests if they take long time to process and prevent recent requests from processing.

![Rate Limiter - Leaky Bucket Algorithm](/img/rate-limiter/rate-limiter-leaky-bucket.png)

- **Fixed Window Algorithm** - As the name suggests, in this algorithm, we fix a time window over which the rate limiting is imposed. Irrespective of the actual time the request came in, the count is maintained for fixed time windows of x seconds.

![Rate Limiter - High Level Design](/img/rate-limiter/rate-limiter-fixed-window.png)

Consider the example in the above image where the rate limit is 3 requests per second, the first request came in after 500 ms. It is clearly visible from the previous example, that from 500ms to 1500 ms, the server actually serves 4 requests this might lead to a rate limiter allowing extra of requests if more such requests come near the boundary of the time window. The next algorithm will overcome this issue with Fixed Window Algorithm.

- **Rolling Window Algorithm** - In the Fixed Window Algorithm, the reference time window was fixed, however in case of a rolling window, the time window starts when the first request is received. The rate limit is imposed based upon the number of requests that came in from the start of the window to the end of the window. This will improve the performance around the boundary of the time zone.

![Rate Limiter - High Level Design](/img/rate-limiter/rate-limiter-fixed-window.png)

As shown in the image above, now the time window starts at 500 ms when the first request comes in, and it allows us to serve only 3 requests R1, R2, R3 for that second. Thus request R4 is throttled, after 1 second is complete, the new time window starts once a new request comes in.

## High Level Design for Rate Limiter.

![Rate Limiter - High Level Design](/img/rate-limiter/rate-limiter-high-level-design.png)

The above diagram shows the High level design for a rate limiting system. When the Webserver receives the request it would ask the Rate Limiter service, whether the request should be served or throttled. If the request is not throttled, it will be forwarded to one of the API servers.

## Distributed Rate Limiter Design

The true benefits of rate limiters are predominant in a distributed system, where we are operating on a large scale, and we want to have efficient use of our resources. In a distributed system, there will be multiple API servers serving requests from users.

Due to the number of requests, we would need to scale our rate limiter service by distributing over multiple nodes. In each of the algorithms, described above we store some kind of count that constraints the number of requests that will be served. How can we achieve this if our counters/buckets are distributed over multiple nodes?

**Communication between the hosts is the key here**. We want to implement the rate limit at a global level, hence we need a way to synchronize among different cluster nodes.

We can use a global data store to maintain the counters for each window and user. But having a global data store is a bottleneck as all the nodes will flock to the data store to get the count. The worst cases could lead to race conditions. If one of the nodes has read the value of the counter before it could update the counter, another node could read the same value and the final value will be incremented by only one instead of two.

![Rate Limiter - Race Condition for global counter](/img/rate-limiter/rate-limiter-race-condition.png)

We could avoid race conditions by introducing locks. However, that could degrade the performance of the system and it will not scale well. Querying the central data store will add the additional overhead of several milliseconds for each request.

**Can we do better ?**
Instead of querying the centralized data store for each request, we can maintain the counters locally. Let's take an example: If our rate limit is 4 requests per second per user. We can have a limit of 4 requests per second per user on each server as shown in examples below.

Global Limit : 4 req/sec/user

![Rate Limiter - Race Condition for global counter](/img/rate-limiter/rate-limiter-distributed.png)

Let's say Server1 received 2 requests in a particular second, Server2 received 1 request and Server3 received 1 request. Since each server is maintaining the counters locally, If 2 more requests come to Server1, it would allow the API call. Thus our rate limiter has served 6 requests per second for the user.

This scheme will lead to relaxed rate limits. However, each node can synchronize the counters with a central data store eventually as a part of the synchronization cycle. If a particular node had served more requests in a time window it would receive a negative counter value for the future time window and hence the average number of requests served per second per user will still be the same as the original limit. We can configure the interval between synchronization cycles to achieve optimal results.

## References and interesting reads.

[Rate Limiting Techniques](https://cloud.google.com/solutions/rate-limiting-strategies-techniques)

[Alternative Approach to Rate Limiting](https://blog.figma.com/an-alternative-approach-to-rate-limiting-f8a06cf7c94c)
