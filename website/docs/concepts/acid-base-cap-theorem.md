---
id: acid-base-cap-theorem
title: ACID, BASE and CAP Theorem
sidebar_label: ACID, BASE and CAP Theorem
---

## ACID

ACID is an acronym for Atomicity, Consistency, Isolation, and Durability coined by Andreas Reuter and Theo Härder as a set of properties of database transactions that every database storage engine should strive for. ACID compliance guarantees the validity of data in events of errors, hardware, or power failures.

**Atomicity** refers to All or nothing which means all the changes which are part of a single transaction are performed or everything is rolled back and none of these changes take effect.

**Consistency** refers to guarantee that all data that is written to the database will conform to defined schema and constraints at the time of saving the data.

**Isolation** refers to the ability of a database to isolate data among transactions providing an independent view of the data. Thus if multiple transactions are executed concurrently, these should not interfere or see intermediate or incorrect data and the result should be the same as if they were run sequentially. Isolation levels are configurable in most DBMS, providing control to Database Administrators to decide the level of isolation. Most DBMS provides the following levels of Isolation: Serializable, Repeatable reads, Read committed, Read uncommitted.

**Durability** refers to the permanent nature of the data that was stored as a part of the transaction once it is successful. Once the transaction is complete the subsequent reads should fetch the last written data.

## BASE

BASE refers to Basically available, soft state, eventual consistency.

**Basically Available** means the system is mostly available and every working node responds to requests in a reasonable amount of time.
**Soft State** refers that the state of a system might vary over time, even without any new input.
**Eventual Consistency** refers to the ability of a system to become consistent over some time. The data across different nodes will reflect the same value.

## The CAP Theorem

The CAP theorem provided by Eric Brewer in 2000 states that a network-shared system can only guarantee /strongly support two of the following three properties:

**Consistency**: Data should be sequential consistent(refers to [linearizability](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf) across all nodes of a distributed system, such that all nodes in a system return same data after a successful write operation.

**Availability**: Every request served by a non-failing node must result in a response in a reasonable amount of time.

**Partition tolerance**: In the case of partition failure, System will continue to work and provide consistent data.

The CAP theorem provides a tool to make design choices while building a distributed system. The CAP theorem is, however, an oversimplification, highly misunderstood, and has received negative publicity over the years. This is because any distributed network shared system is inherently prone to Partition failure. One should always try to tradeoff between consistency or availability in case of partition failure. Also in modern distributed systems, the notion of Availability is dependent on the latency of the system. It is not acceptable to call a system available if it takes more than 100 seconds.

Here is an interesting [article from Martin Kleppmann](http://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html), on how CAP theorem uses very narrow definitions that cannot describe modern distributed systems.
