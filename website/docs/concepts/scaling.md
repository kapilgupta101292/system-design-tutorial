---
id: scaling
title: Scaling
sidebar_label: Scaling
---

Scaling means that increasing the system capacity to cope up with increased load for example number of requests or handling more data. 

## Need for Scaling

if our website is currently handling 100 requests/min and suddenly the traffic increases to 1000 requests/min, even though our system might be able to handle the same the traffic but due to increased load, the performance might become unacceptable, hence we will need to scale our system to be able to support that additional load with acceptable performance.

## Type of Scaling

### Vertical Scaling (Scaling Up) - Increasing the capacity to the current machine by improving its specifications like attaching more RAM, adding better CPU, upgrading HDD to SSD or adding more storage capacity, increasing network bandwidth etc.

#### Advantages: 
Architecture - Relatively simple, Less issues to worry about like inter service dependencies, network partitions etc.
Maintenance - Easy to maintain due to less moving parts.

#### Disadvantages:
Cost - Adding more resources leads to exponential increase in cost.
Scope - Limited scope of Scaling due to physical limitation of size.
Downtime - Risk of fault or complete downtime due to hardware failure. 

### Horizontal Scaling (Scaling Out) - Adding more machines and distributing the load among them. 

#### Advantages: 
Cost - Cost efficient due to the use of commodity hardware.
Scope - Theoretically unlimited scope of scaling.
Resilience and Fault Tolerant - Due to redundancy, if few nodes go down, other nodes take up the requests.

#### Disadvantages:
Architecture - Complex to design, due to fallacies of distributed systems.
Maintenance - Difficult to observe and maintain due to moving parts.

### Which approach is the best ?

There is no single correct answer to this, the choice of how to scale a particular system depends specifically on our system. The bottleneck for the system may be the number of requests, the volume of data , the access patterns or something else. 
Depending upon whether you are targeting sub 200ms response time on your API’S or trying to manage petabytes of data, The System design and scaling strategies will be entirely different.
It is not possible to find an approach that fits all. 

In general, it is easy to distribute a stateless system across multiple machines. Managing and synchronizing state within shared systems can introduce a lot of additional complexity and becomes hard to design and maintain. 

It is a good practice to use a hybrid approach. To keep it simple, We can first scale vertically by using relatively powerful machines till we reach a point where it makes sense to use a large number of smaller machines. 

Also, it is an ongoing process, a system designed for a particular load may not be able to handle 5x or 10x the load. At each milestone, we need to optimize, monitor and constantly re-architect on every magnitude of increased load.
