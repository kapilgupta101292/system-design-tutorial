URL shortening Service creates a compact version of a long URL called tinyURL or shortURL. These URL's can be shared as an alias to the long URL and when someone hits this shortURL they are redirected to the original long URL. Benefits of short URL includes more readablity, convenience to the user and save space. They are much more user friendly when displayed on screen, shared, or tweeted.

URL shortening is also used for tracking and analytics purposes for performance measurements of different campaigns and affiliates. Some URL shortening services also allow yout to provide custom domains to promote your own brand.

Example:

```javascript
https://www.google.com/imgres?imgurl=https%3A%2F%2Fnationalzoo.si.edu%2Fsites%2Fdefault%2Ffiles%2Fnewsroom%2F649a1243-cropped.jpg&imgrefurl=https%3A%2F%2Fnationalzoo.si.edu%2Fanimals%2Fnews%2Fcelebrating-national-panda-day&tbnid=ExJDq27RAbY-3M&vet=12ahUKEwiMnrW6pqbqAhUyFrcAHeVmDxsQMygDegUIARDYAQ..i&docid=WNx1NKHAfQ_wJM&w=4819&h=2410&q=panda%20image&ved=2ahUKEwiMnrW6pqbqAhUyFrcAHeVmDxsQMygDegUIARDYAQ
```

can be shortened to this

```javascript
https://bit.ly/3i8Yitn
```

## Requirements and Goals of the System

Requirements for a URL Shortening service.

### Functional Requirements:

- Create a unique short URL for any given URL.
- When user hits the shortened URL, he should be redirected to the original long URL.
- Shortened URL must exprire after a configurable expriation time.
- Based on the subscription user should be able to pick up a custom domain for the links to promote their own brand.

### Non-Functional Requirements:

- Our redirect service which redirects any short URL to the original URL must be highly available. Otherwise users won't be able to access the required URL.
- Redirection should have minimum latency otherwise it will be a bad user experience and users might switch back to long URL.

### Good to have features

- Our service should provide analytics on the usage of shortened URL, like the total number of times it was hit, countries where it was hit, etc.
- We can also expose our services throught REST endpoint to allow programmatic access to developers, supporting bulk requests.

### Estimations

**Traffic estimates** :

We can assume that we will have around 10M new URL shortenings per day. There will more number of redirections to the original URL than the creation of new shortened URLs. Hence let's assume a 1:100 write to read ratio.

```javascript
Total number of new URL shortened per day = 10M
URL Shortening Queries per second = 10M/(24 * 3600) ~= 40M/(100 *3600) = 120QPS

Total number of redirection requests per day = 100 * 10M = 1B request/day
URL Redirection Queries per second = 100 * 120 QPS ~= 12K QPS

```

**Storage estimates**:
Assuming that we 10M new URL shortened per day,

```javascript
Total number of records stored per day = 10M * 365 * 5 ~= 10M*400*5 = 20 Billion

Assuming 1 KB storage space per record,
Total memory required to store the data = 20B*1KB = 20TB

Assuming, 20% of the total read requests are cached
Total cache memory required = .2 * 12K *3600 *24 *1KB = 200GB

```

## API's

Our service can expose the following REST API for creating and deleting shor URL's.

```javascript
generateShortURL(String longURL, String customDomain, String apiKey):
```

1. **longURL(String)** : The URL which needs to be shortened.

2. **Custom Domain (String)** : Custom domain to be used for shortened URL. This could be a limited feature based upon the subscription of the client.

3. **apiKey(String)** : The API key for the client making the API request, used for access check, rate limiting, and analytics purpose.

API will provide a JSON response with the generated short URL.

```javascript
deleteURL(String shortURL, String apiKey):
```

1. **shortURL(String)** : The URL which needs to be deleted.

2. **apiKey(String)** : The API key for the client making the API request, used for access check, rate limiting, and analytics purpose.

API will provide a JSON response with appropriate status if url was deleted.

## Database

We need to store billions of records over the time. Also our service is read heavy.

What about the consitency requirements? this is a very simple use case where we only create a new record for the shortened URL and then never update the same record. Hence we don't have strict consitency requirements.

What about the availability requirements ? Our service should be highly available otherwise users will not be able to access the original URL and will result into loss of business. Hence we want strong availability.

The trivial choice will be to use a NoSQL database like Cassandra, MongoDB or DynamoDB because we need a database that can easily scale. We can store the URL's in the following collection. To retreive the URL fast we can add index no the hash column.

```javascript

   //URL Collection Sample Document:

    {
        "_id": "507f191e810c19729de860ea",
        "hash": "3i8Yitn",
        "original_url": "https://www.google.com/imgres?imgurl"
        "creation_date": "2020 Jul 11 16:54:20 UTC+5:30",
        "expiration_date": "2021 Jul 11 16:54:20 UTC+5:30",
        "user_id": ""
    }

    // User Collection Sample Document:
    {
        "_id": "507f1f77bcf86cd799439011",
        "name":"John Doe",
        "subscription_tier": "",
        "creation_date": "2019 Jul 11 16:54:20 UTC+5:30"
        "last_login_date": "2020 June 11 16:54:20 UTC+5:30"

    }

```

## High Level System Design

### Approach

Since we want to convert a long URL to a short one, we can choose to use 6-8 characters to represent our tinyURL. Let say we have a character set of 64 (0-9, a-z, A-Z , -, .), and
We can have a total of:

```javascript
    64^6 ~= 68 billion
    64^7 ~= 4.39 trillion
    64^8 ~= 281 trillion
```

Based upon our estimations, we are expecting around 20 billion URL's over the 5 years, we can go with 6 characters to represent our tinyURL.

The Naive approach to convert the URL will be to maintain a counter, increment that counter every time we have a new request for tinyURL. However this scheme is not suitable for a scalable service. Several servers will compete to get the new value of the counter and it will become a bottleneck.

Another approach could be to generate a random number, this approach could scale as different servers could generate random numbers in parallel. In Practical scenarios there is no possibility of collisions if we use the correct random number generator. We can use a UUID generator. However it will have 36 characters, if we take only first six characters, we can increase the chance of collisions. In such scenario we can check whether the first six characters are already used. In that case we can use the next 6 characters and so on until we get a unique tinyURL.
The problem with this approach is that if you make several requests for converting the same long URL, it will result into a different tinyURLs.

To overcome above problem, we could use any popular Hash Algorithm like (SHA-2 or MD5) to generate a hash of the original URL. We can encode the hash value to the base 64 and use the first six characters of the hash value. This way we would always generate the same hash value for different requests for the same long URL. But this approach could also generate duplicates, we can use the same approach descibed above by checking if the tinyURL is already used and belongs to the same long URL. If so, we can use the next six characters and repeat the process.

### Scaling the Key Generation.

The problem with the above approach will be that the key generation generation can become a bottleneck as we scale our service. It would be time consuming to check whether the required key is already present in the database.

We can overcome this limitation by generating the keys in advance using. We can have a dedicated key generation service that will create a range of keys in advance. This range will be distributed across multiple nodes and will be loaded in memory, hence no processing time will be spent in hashing or encoding the URL.

Since each server has its own unique set of keys, we don't need to worry about any duplicates or coordination between the servers. When the request comes in to shorten a particular URL, the server will pick one of the keys and assign it to the URL, by making an entry into the database. It will then mark the key as used, or remove it from the memory.

**What happens if a server dies ?** If the server dies, the range of keys will be lost. However that is absolutely fine, since we have huge number of keys.

**What happens if the Key Generation service fails** ? Since we are now depending upon on the key generation service, it should have replicas to make it highly unlikely available in event of node failure.

## URL Redirection

When the User enters the tinyURL in the browser, the request will hit our API, API will fetch the data from the database based upon the tinyURL ke. If the key is not present user will be shown a "404 Not Found" HTTP error, other wise, user will be redirected to the original URL that was shortened.

### URL Expiration

Since we need to store data for the shortened URL's, we can choose to clean up URL's which were created long back based upon a certain expiration date and the frequency of usage. This expiration date could depend on subscription of the user who created the URL. This will clean up the data

For the Free tier users, we can have an expiration time of 6 month or 1 year from the last used time, before the data is purged. Similarly we can have a longer expiration time for paid users according to their subscription plan.

We can run a scheduled job every 24 hours that will cleanup the expired URL's from the database. The service should run at time when user traffic is low to prevent load on the database. For example, we can choose to run the job at midnight in each region. We can also reuse the key from the expired link.

## Data Partitioning and Replication

In order to scale our DB, we need to introduce some partitioning scheme to store URL's. This will distribute our load to different database servers.

We can do one of the following:

- Partitioning based upon Range: We can do a range based partitioning by distributing URL's across partitions based upon the first letter of the URL. Although this is a simple approach, but it can lead to unbalanced partitions. If we have a lot of URL's that start from 'G', the server that stores these URL's can becom a hotspot.

- Partitioning based upon Hash: Under this scheme, we can distribute the URL's among different DB partitions based upon a hash value. If our hash function can ensure uniform distribution the partitions will be balanced. We can use consistent hashing to avoid any problems related to uneven loads.

## Caching

Since our application is read heavy, we must use some caching strategies to store the hot URL's.
We can assume that almost 20% of the URL's will contribute to 80% of the traffic. We can store a mapping of the shortened URL and the original URL.

To store 20% of the hot URLs, we need to (200GB) of memory. We can use any popular caching system like Redis, or Memcached. We can create a distribute caching system to scale our caching layer using more servers.

Caching the server will reduce the load on the DB servers and improve the overall performance of the system.
As discussed in [Caching](caching) tutorial. We can use a suitable cache eviction policy like Least Recently Used (LRU).

Similarly we can use a suitable cache write policy to update the cache in case of a cache miss. We can use the write around cache policy, initally data will be written directly tothe database while bypassing the cache. While reading recent data there will be a cache miss and will result in data being read from the disk. At this point data will put into the cache.
