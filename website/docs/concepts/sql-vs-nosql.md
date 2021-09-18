---
id: sql-vs-nosql
title: SQL vs NoSQL Database
sidebar_label: SQL vs NoSQL Database
---

A key aspect of any Large scale system is the ability to handle a large amount of Data.

A Database is used to persist information that will be useful later on. Broadly there are two types of database categories available to store the data and those are SQL or NoSQL. Which one to use depends on the needs of the system. For many years in the past SQL or Relational databases were the standard to store information. As the systems became more and more complex and the data grew many-fold NoSQL databases have gained their ground over the last decade and emerged as the primary data store for many applications. To understand which database is better for your needs to let's understand the difference between these two.

## SQL or Relational Databases

A SQL or Relational database stores data in an organized set of tables with rows and columns. All the data related to a particular entity is stored in one table or more tables. A row stores all the related values for a particular instance of entity or object and each column stores one attribute of that object. Queries data using SQL syntax and JOINS. CRUD uses SCHEMA and transactions.

## NoSQL Databases

NoSQL Database refers to Non Relational database, In contrast to a relational database where data is stored in well defined tabular relations and Structured Query Language is used to access data, NoSQL database uses different mechanisms for storage and retrieval. Different flavors of NoSQL use proprietary storage methods. Based on the problem they are solving NoSQL Databases store data in a wide variety of forms like key-value pairs, like documents, in columnar form, in the form of graphs or special time-sequenced events.

## SQL v/s NoSQL

### Schema and Structure:

**SQL:** Data is nicely organized in appropriate tables according to the predefined schema, which reduces redundant information. Data conforms to the applied constraints. Requires a lot of initial thought to minimize changes later which might require migration of existing data and related downtime. Structured data prevents developers from sloppily adding data, and constraints prevent the corruption of data due to software bugs. This translates to more effort for developers.

**NoSQL:** schema is flexible, depending upon the type of NoSQL database, we might add or skip different columns and data in each document. It provides the flexibility of the data model to the developers allowing for easy iterations through the development process and no downtime.

### Data Retrieval:

**SQL:** Due to structured data, the Relational database provides a powerful SQL(Structured Query Language) interface for Data definition, Data Control, and Data Manipulation. The split structure allows us to join data in any way. Numerous join types are available and can be done with any number of tables in any way.

**NoSQL:** Data is stored in a nested form with everything related to an entity stored in a single document. Though it introduces data redundancy, but provides very fast access and allows horizontal scaling. Depending upon the type of NoSQL certain queries are extremely fast, for example, the Key-Value database provides fast lookup, Column based database provides fast aggregation capabilities.

### Scalability:

**SQL:** SQL Database is powerful and flexible but constrained in terms of scaling. Scaling SQL Database systems require expensive hardware to scale vertically. If we distribute SQL over multiple servers using data partitioning, however, performance suffers for certain queries and joins.

**NoSQL:** These are built for web-scale applications and are easily scalable horizontally. Each type of NoSQL database is optimized for certain queries. NoSQL does not use custom expensive hardware that allows for faster retrieval of data at high scale. Almost all popular cloud vendors provide managed solutions that scale very easily according to growing traffic.

### CAP Theorem:

**SQL:** Majority of SQL Database offers ACID compliance and ensure consistency and reliability of data that is stored. SQL database provides consistency and availability using clustered services.

**NoSQL:** These are majorly designed for scale and hence focus on Availability and Partition Tolerance by compromising on the consistency of the data.

During a design interview, it is of utmost importance to decide the right database for storing data as it will allow you to scale and optimize your system. We need to choose the database that is the best fit for data, the type of data will drive that selection.

For example: If you are designing a Banking application or other financial services related application, It is no brainer to choose a Relational Database because of the data guarantees and security it has to offer which is a prime concern for this use case.

On the other hand, If you are designing a web crawler or search engine application, it is not advisable to use a Relational database due to the unstructured nature of web data and the scale at which you need to operate.

Here is a list of Popular Use cases along with the most suitable Database types for each use-case.

| Sno | Use case             | Choice of Database                          |
| --- | -------------------- | ------------------------------------------- |
| 1   | Cache                | NoSQL inmemory datastore : Redis, Memcached |
| 2   | Banking Application  | SQL Database                                |
| 3   | Social Network       | NoSQL Graph Database: Neo4j                 |
| 4   | Facebook Ad Platform | NoSQL columnar Databse: Cassandra, BigTable |
