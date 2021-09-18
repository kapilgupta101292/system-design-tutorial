---
id: nosql
title: NoSQL Database
sidebar_label: NoSQL Database
---

NoSQL Database refers to Non Relational database, In contrast to a relational database where data is stored in well defined tabular relations and Structured Query Language is used to access data, NoSQL database uses different mechanisms for storage and retrieval and generally does not provide SQL based data access and hence the name "Non-SQL", Although some of NoSQL implementations may provide SQL kind of interface, hence NoSQL is also sometimes called as "Not Only SQL".

NoSQL Database doesn't use relational tables or schema. Different flavors of NoSQL use proprietary storage methods. Based on the problem they are solving NoSQL Databases store data in a wide variety of forms like key-value pairs, like documents, in columnar form, in the form of graphs or special time-sequenced events.

NoSQL Database is inherently designed for Availability and Partition tolerance over Consistency, however, different vendors do provide the support to increase the level of consistency by trading off other attributes.

NoSQL database has gained a lot of popularity in the past decade as it is designed for overcoming limits of scale and providing the ability to scale horizontally. These are created by large web companies and a lot of popular ones open-sourced. NoSQL database is highly prevalent in the Cloud Data storage model. Major cloud vendors like AWS, GCP, and Azure provide managed NoSQL solutions with enterprise-level support and tooling.

## Characteristics of NoSQL

- **Scale**: NoSQL databases can store data on very large scales. These are designed and optimized for specific uses cases at a super large scale, something that cannot be achieved by the relational database. This helps in scaling your data storage to such extent and allowing fast retrieval for specific queries. NoSQL database is used for solving problems like indexing the web for search engines like Google and Bing, predicting customer behavior for analytics and ads on platforms like Google and Facebook or backing recommendation systems for Netflix.

* **Flexibility and Ease of Use**: Another reason for widespread popularity is the flexibility of use, Due to flexible schema, these are best to get started without putting much thought about the structure initially.

* **OpenSource and Community Driven**: Most of the popular NoSQL database are open-sourced, and provide community editions of these products.

## Types of NoSQL Database

Following are the popular types of NoSQL database:

- **Key-Value NoSQL**: These databases store huge lists of key-value pairs, where the key represents the **field or attribute** name, and the value represents the **value** of that field. These store hot datasets mostly for caching or lookup purposes. They provide extremely fast access through in-memory storage options. Most popular key-value stores include DynamoDB, Redis, Memcached, and Voldemort.

- **document-oriented NoSQL**: Data is stored in the form of documents, the documents are further grouped in collections. In contrast to the rows in a Relational Database, where each row stores data for a fixed number of columns, The structure of each document in Document-Oriented DB is flexible and does not need to be the same as other documents. These databases are fast for querying and easily scalable, Most popular Document Datastore are MongoDB, AWS DynamoDB(can act as both key/value and Document) CouchBase, and Elasticsearch.

- **Columnar NoSQL** : Data is stored in columnar families, unlike conventional database this database allows us to read specific column values. Best used for storing ragged datasets, for purpose of aggregation. These are extremely fast and scalable and suited for analytical applications. Most popular Columnar databases are Cassandra by Facebook, BigTable, and BigQuery by Google Cloud Platform.

- **Graph NoSQL** : Data is stored in the form of graph bases structures to store entities and their properties. These are extremely fast for relationship queries and easily scalable. Examples include Neo4j and AWS Neptune DB.

- **Speciality NoSQL** : These are specialty databases created and optimized for certain special types of data like Time based events, IoT events, and blockchain ledger data.
  Example of this type of database includes Google Cloud Platform Firebase/Firestore, AWS IoT, and AWS QLDB.

## Trends in NoSQL Landscape

As NoSQL database are becoming widely used, different vendors are adding multiple features within the same offering, like the AWS DynamoDB can act both as a Key-Value and Document Database. Redis has support for in-memory and persistence on the disk. New features are getting added for elastic scaling, backup, and monitoring.
