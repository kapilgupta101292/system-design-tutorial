---
id: caching
title: Caching
sidebar_label: Caching
---

Caching means storing data into the cache, which is a high-speed low latency data storage. It will store a limited amount of information, that is very likely to be used in the immediate future. Caching layer typically stores the data in memory (or RAM) and hence avoids accessing the secondary memory (disk) which is an expensive operation in terms of time. Caching allows optimal use of available resources by using the principle of **locality of reference**, according to this principle processors tend to access the same set of memory locations repetitively over a short period of time.

### Caching Layers

In modern large scale systems which serve billions of request, depending upon the needs caching layer may exist at multiple levels like at the front end between the load balancer and application server to cache response data of frequent requests, or between the application server and the persistence layer to prevent costly disc access.

## Caching at scale

It is easy to maintain and use cache for small scale applications where a single node is applicable, but what happens when we scale the application to multiple nodes. We have the following options:

### Global Cache

This is a simpler option where we maintain a single global cache space, which is used by all nodes. For every request, the application queries this cache to retrieve data. A global cache is simpler and effective up to a certain scale, however, it could become a potential performance bottleneck and point of failure, if the number of requests increases beyond the maximum capacity of the cache. This is where the distributed cache comes in.

### Distributed cache

For a distributed system application, we need a distributed cache. We distribute the cached data to multiple nodes. When a request comes in, a lookup happens to determine on which machine the data is cached and then that node fulfills the cache request.

A distributed cache is very effective and performant and capable of handling requests for very large scale systems, however, it is more complex to maintain, as it has all the intricacies of a distributed system.

Examples of Cache systems used in Distributed Systems:
Redis
Memcached
Aerospike DBS
Apache Ignite

### Cache Write Policies

Before reading the data from a cache, we need to write it to the cache when the data is first requested so that we can later serve the data in less time. The following policies are used to write and maintain data in the cache.

--**Write through cache** : As the name suggests, In this scheme data is written to a database through a cache, and write operations are confirmed only if write to the database as well as cache are successful. This ensures strong data consistency between cache and persistence layers but makes the write operation costly with respect to time due cache write overhead.
This scheme is suited for systems which are read-heavy and have much fewer write operations than read.

--**Write around cache** : In this scheme, data is written to the database while bypassing the cache. Hence write operation is faster. However, while reading recent data there will be a cache miss and will result in data being read from the disk.

The write-around scheme is good for use cases where the recently written data is not read frequently.

--**Write back cache** : In this scheme, data is written only to the cache, and then write operation is confirmed to the client. This results in blazing fast write operations as we don't write the data to the database. Subsequently, data is asynchronously written to the database in a separate thread. All read requests are served from the cache hence strong consistency is maintained.

This scheme is well suited for write-heavy applications, However, this poses a huge risk of data loses in case cache goes down, though the risk of data loss can be minimized through replication across multiple nodes in a distributed system.

### Cache Eviction policies

Since caches can hold only a portion of data, we need to discard the data that is no more useful, How do we determine what data is useful and what is not? Following are the most popular cache eviction policies:

--**Least Recently Used (LRU)**: Removes the items which have least recently used or in other words the items which were used recently are kept. Most popular eviction policy as it provides good performance while maintaining simplicity.
--**Most Recently Used (MRU)**: Opposite of LRU, this policy discards the most recent items first.
--**Least Frequently Used (LFU)**: Discards based upon the frequency of items and items with less frequency are evicted in comparison to more frequent ones.
--**First In First Out (FIFO)**: One of the simpler strategies, where the item which was used first goes out first, like in a queue.
--**Last In First Out (LIFO)**: The items which were used last go out first, like a stack.
--**Random Replacement (RR)**: This eviction policy discards a random item.

Besides these classical approaches, there are some modern eviction policies that provide better hit rates and concurrencies like LIRS, Window-TinyLFU, and ARC. More information can be found at this [link](http://highscalability.com/blog/2016/1/25/design-of-a-modern-cache.html)

### Content Delivery Network (CDN)

Content Delivery Networks are powerful systems distributed geographically, which are mostly used to cache and serve static content and media for any application. Requests are first sent to CDN for any static content and served from there. Only in case of a miss request will be directed to the application servers. Due to the distributed and powerful networks, CDNs today serve a majority of web traffic and allows blazing-fast transfer of static media.
