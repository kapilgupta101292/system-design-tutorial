---
id: yelp
title: Yelp System Design
sidebar_label: Yelp System Design
---

**Yelp** is a platform for crowd-sourced local business reviews and listings. Locations like restaurants, bars, and other businesses have dedicated pages, where users can read or submit reviews and ratings. Businesses can provide information about themselves on their pages so that people can discover their services and timings. In addition to reviews, Yelp also provides the ability to upload images and videos related to a location.

Users can search for any place or nearby attractions like restaurants, theaters as well as events. Other similar services include TripAdvisor (for travel), Zomato (for Restaurants)

## Functional Requirements

1. Business Profile - Yelp provides dedicated pages for each business, business owners or customers should be able to create a page for Businesses.
2. Search by Location or GPS coordinates - Users should be able to search for attractions by providing location and also able to search nearby places.
3. Reviews and Rating System - Users should be able to provide reviews for the places they have visited, services they have availed. Users should be able to provide ratings in terms of stars out of 5.
4. Photo and Video upload - Users should be able to add photos and videos related to the place.

## Non Functional Requirement

1. High Availability: Our search service should be highly available.
2. Scalable: There will be peak demand on the Holiday season or certain tourist attractions.
3. Low Latency: Users should be able to access information as fast as possible.

## Additional Requirements

1. Fake Reviews Detection: The system should be able to detect fake reviews by business owners that try to increase their ratings unethically.
2. Image and Video Processing: These services process the images and videos uploaded to provide the best photos to its users.

## Estimations

We need to store location entries, Assuming that we will be storing around 500M places. Let’s also assume a 20% growth in the number of places each year, we can safely assume that we don't need more than 1MB(excluding photos and videos) to store the metadata related to a location.

Our system will be read-heavy, we can assume a 1:1000 write to read ratio with around a peak of 100k concurrent users.

## Database Schema

We can store the following data in a relational database. We can have a place/location table to store the place information

**Location Table** :

```sql
id : integer
name : varchar(256)
geohash : varchar(12)
description: varchar (512 bytes)
address : varchar (1024)
category_id: integer
```

Similarly, we will have tables for users, reviews, ratings, photos, and videos. We can store the photos and videos on the cloud blob storage like AWS S3 and maintain a reference in the photo and video table respectively.

## API's

Our service can expose the following REST API for searching nearby places.

```javascript
search(String searchTerm, Map<String, Object> search params, String apiKey):
```

1. **searchTerm(String)** : The term which the user is searching for, it could be the name or location of the business.

2. **search params(Map)** : search criteria, includes the location for which search needs to be performed, the search radius, additional filter params like categories, number of results to return, sorting order, etc.

```javascript
    sample searchParams :

    {
        "location": [34.3, -118.243],
        "categories" : ["restuarent", "bar", "dine-out"],
        "searchRadius": 5,
        "maxResults": 100,
        "sortCritera": 1,
        "sortAscending" true,
    }
```

3. **apiKey(String)** : The API key for the client making the API request, used for access check, rate limiting, and analytics purpose.

API will provide a JSON response for the list of places matching the search criteria.

We can expose additional API to Add/Modify new listing, add reviews, add ratings, upload photo/video to the listing, or a particular review.

## High-Level Design

At a high level, The key feature of the Yelp system design is to find the nearby places/businesses with minimum latency and sort them by distance, because everyone wants to do business with the nearest store.

How to store the geographical context in the database.

**Naive solution** - A naive solution will be to store latitude and longitude data for every POI, same for the user, and then compare the information of the user's location with the database. Given a certain range query, the database where latitude is between xi and xj and longitude is between yi and yj.

This approach will not be scalable given the huge number of places, that users can query for.

**GeoHash**: Sequence of characters that identify a particular region to design a scalable system that provides geolocation data, we can use the concept of Geohash.

GeoHash essentially encodes the latitude and longitude information about a place to a String. The world map is divided into a rectangular grid system. Each rectangle is identified by a hash string and is further divided into 32 rectangles following a hierarchical structure, each level having an extra character than the hash of the parent rectangle. The "Geohash alphabet" (32ghs) uses all digits 0-9 and almost all lower case letters except "a", "i", "l" and "o".

![Geohash](/img/geohash.jpg)

Geohash represents a boundary and not a point. Since our mission is to calculate the nearby places, it is well suited for the application. Also, there will be some corner scenarios that we need to consider while determining the neighbors as shown in the below picture.

![Corner Cases in Geohash](/img/geohash-corner-case.png)

Geohashes are popular for spatial indexing and search applications thought initially were used only as part of URL-shortening service. If you have never heard the concept behind **Geohash**. I highly recommend viewing the following video.

https://www.youtube.com/watch?v=UaMzra18TD8

Let's return to our Yelp System Design scenario for finding nearby places and restaurants.

The naive algorithm to find out the nearest restaurants and businesses would be to find out the points with the same geohash as the location of the user. As discussed above due to corner cases, some of the nearby points of interest, but depending on the distance we want to search, likely leave out nearby points of interest that are in neighboring geohash bin.

To get correct and a more comprehensive set of points of interest, we have to get all the points in the current bin as well as all the points in the surrounding geohash bins i.e 8 immediate neighboring bins. Then we calculate the distance between all the points around the center.

Using Geohash based approach to find a point on the map, we can handle a very large dataset since on entering a particular geohash bin, we are essentially discarding the remaining bins, reducing the size of the problem exponentially. Users of a Yelp like service needs to see the results in real-time, hence we need to store and index the data about geohash of the places and associated reviews and ratings. Since the location data will not change, we need to implement the indexing for reading efficiency.

We can use a reasonable Geohash size of 12 characters to represent any place on earth with good precision. We can store the Geohash for locations in the database.

![Geohash Precision](/img/geohash-vs-area.png)

### Storing GeoHash

Let’s see what are different ways to store this data and find out which method will suit best for our use cases.

#### Custom Approach

Since this is hierarchical data, we can store the data in the form of a tree structure. Each node will represent a **geohash** and will 32 children for each of the nodes to model the geohash like structure.
We will have additional tables to store the metadata for any place corresponding to each geohash.

**Building the tree structure**

We will start with the root node that would represent the whole map. The root node will have 32 children. We will insert take the 12 character geohash of the place we want to insert into the tree and will create a node for each character. For example to insert a place represented by the geohash "9q8zhuyj1ccb1" we would create a node under root node to represent the grid "9", under that node we will create a child node to represent "9q" and so on.

**Querying the grid for a place**

We can start from the root node and search downward for the required node. At each node, we will move towards the child node with the next character in the geohash, Once we reach the leaf node, that is the required node representing the point of interest. If at any point we don't find the required node, then the point of interest does not exist in the grid.

**Finding out the neighbors**

To find out the neighbors of any point, we can go to the parent node, or grandparent node depending upon the radius for which we want to search for neighbors, find out all the points under them, Calculate their distance from the Point of Interest and then sort according to the given criteria.

We can optimize the data structure for finding the neighbors, by connecting the leaf nodes with a double linked list to allow quick forward and backward iterations among the neighboring places.

After getting the nearby geohashes, we can query our metadata table to find the details corresponding to those places.

**Memory Requirements**
Since each leaf node represents a place using a Geohash of 12 characters, Total memory needed to store the Tree Structure for 500M places will be :

    12 * ~1Byte * 500M = 6GB

Since we also have internal nodes in the tree, assuming around 1/3 internal nodes with 32 children, they will occupy:

    500M * 1/3 * 32 * ~1 Byte = ~5GB

### Use NoSQL offerings like Redis, ElasticSearch, MongoDB to store the Geospatial data

Instead of writing custom logic to implement Geospatial search, we can use some popular NoSQL databases like Redis, Elasticsearch to achieve the same functionality.

For example, We can use the Geospatial indexing feature of the Redis database. to quickly implement this feature offloading the indexing, searching, and sorting work to Redis using very few lines of code.
We can use the Geo Set to work with spatial data in Redis, Redis provides commands like GEOADD, GEODIST, GEORADIUS, and GEORADIUSBYMEMBER.

More information can be found using the below linked
[Working with Geospatial Data in Redis](https://www.infoworld.com/article/3128306/build-geospatial-apps-with-redis.html)

Similar support is provided by ElasticSearch and MongoDB databases. Going with elastic search gives full-text search as well as we need to search by the geographic tuple.

[Working with Geospatial Data in ElasticSearch](https://medium.com/@yatinadd/going-geospatial-with-elasticsearch-using-geo-points-plus-its-application-b013c638064e)

[Working with Geospatial Data in MongoDB](https://docs.mongodb.com/manual/tutorial/geospatial-tutorial/)

## Data Partitioning

If we are using NoSQL offerings like Redis, MongoDB, Elasticsearch, etc. We can easily scale horizontally as we scale and add new places.

If we are building a custom application using the Tree-based structure. We can use the following schemes for Partitioning our data.

- **Partitioning based on regions**: We can divide the complete tree into multiple parts based upon the region. Places in the same region will be stored on the same node. Before storing a new location, we need to first find out which nodes store the data for a particular region, and then we can store a new location on those nodes, Similarly to querying the data.

  The problem with this approach is that there will be certain regions like in San Francisco which will be densely populated and will have more restaurants and other Points of Interest. This will result in an uneven distribution of data and would cause more loads on the server than others. To prevent this problem, we might partition these hot regions further or use consistent hashing.

- **Partitioning based on Hash**: We can use consistent hashing to decide which server will hold the data of a region, we can use a hash function to hash the geohash of a place and then distribute the data uniformly across multiple servers.
  to fetch all the nearby places, we will get the data from all servers and aggregate the data.

**Processing the photos and videos** Yelp stores the photos and videos and applies deep learning models to enhance them on the go.
More information can be found on the Yelp Engineering Blog.

[Yelp Engineering Blog](https://engineeringblog.yelp.com/2016/11/finding-beautiful-yelp-photos-using-deep-learning.html)

**Spam detection** : Similarly Yelp like services uses machine learning models to prevent false reviews and ratings.

## Caching

As our service is read-heavy, there will be around 20% of places which are searched frequently. We can use a caching solution like Memcache or Redis to store this data. This will improve the performance of 80% search queries.
To deal with hot Places, we can introduce a cache in front of our database. We can use an off-the-shelf solution like Memcache, which can store all data about hot places. Application servers before hitting the backend database can quickly check if the cache has that Place. Based on clients’ usage patterns, we can adjust how many cache servers we need. For cache eviction policy, Least Recently Used (LRU) seems suitable for our system.
