---
id: design-twitter
title: Design Twitter
sidebar_label: Design Twitter
---

**Twitter** is one of the largest social networks where users can post and read tweets ( short text messages with 140 character limit) along with photos and video support.  
Users of the Twitter service can also follow their family, friends, or celebrities to get the latest updates about them.
In Twitter, there are two types of timeline features, 
User Timeline is a list of all tweets that the user has tweeted.

Home Timeline or News Feed -> is the temporal merge of the user timelines that you follow with certain business rules around it. (complex logic and multiple signals used for this and settings like don’t see the retweets from a particular user etc.)


## Requirement Gathering

- Publish Tweets: The user should be able to tweet and see the news feed on his/her home page
- News Feed: The user should be able to see tweets/posts from the users that she follows.
- Relevance: The home timeline/newsfeed should be relevant and sorted in reverse chronological order.
- Follow/Unfollow: The user can follow/unfollow other users
- Notifications: The user should be notified about the latest events that are relevant to him.
- Search: The user can search for tweets, hashtags, people.

Apart from the basic features above, we can support the following requirements.

- Trending Topics
- Retweet
- Comments
- HashTag support for posting and searching for tweets.
- Recommendations.
 
Above are the functional requirements of the Twitter Design, Here are some non-functional requirements

- High Availability - Service should be always up to allow users to post and read the tweets.
- Latency - Users of a service like Twitter rely on it for quick updates on what is happening in the world, hence the goal is to minimize the latency. It is unacceptable for Twitter for messages to reach the followers in more than 5 sec time frame.
- Eventual Consistency - We are ok if a particular tweet is available to one user before another.  


## Challenges

**Read Heavy System**: In the case of Twitter, we have a read-heavy system, since the number of reads(timeline requests) is much more than the number of writes(post tweets). We need to keep the latency as minimum as possible so that the updates reach the followers in time.

**Tip**: Whenever we are designing a read-heavy system, we need to make efficient use of caching and precomputing, which will help us reduce the latency to a minimum.

## Estimations

Total Number of Users on platform -> 1 Billion
Total Number of Daily Active Users -> 100 Million
Total Number of New Tweets / Day -> 150 Million
Total Number of Follows per user -> 200
Total Number of Timelines generated per day -> 100 Million * 5
Total Number of Tweets viewed per day -> 100 Million * 5 * 20 tweets = 10 Billion

Storage: 

Storage required to store 1 tweet -> 140 char * 2 Bytes / char  + 20 Bytes for Metadata -> 300 Bytes
Storage required for storing tweets per day -> 300*150 Million = 45000 Million Bytes = 45 GB 
Avg Storage required for one photo -> 1MB
Avg Storage required for one video -> 15MB

Assuming 1/10 of tweets contains one photo and video 
Storage required for photo and video -> (1 + 15 ) MB * 150 Million /10 => 240 Million MB -> 240 TB/day

Bandwidth Estimates

Total ingress -> 240 TB/day -> 2.9GB/sec
Total egress -> 10 Billion * 300 Byte + 10 Billion /10 * 1 MB + 10 Billion / 10 * 15 MB ~= 185 GB/sec 

## API's
Our service can expose the following REST API for posting a tweet and getting the news feed

```
POST /users/{userId}/tweet
{
	“api_key” : "test_key",
	
	“user_id” : ‘2244994945’”,
“tweet_text”: “My First Tweet”,
	"entities": {
       "media": [
          {
          "id_str": "1494379920126095365",
          "media_url": "https://pbs.twimg.com/media/FL0anqqXsAU-KDH.jpg",
          "type": "photo"
        }
      ]
    }
}
```

api_key(String): The API key for the client making the API request, API Key is used for checking the access, rate-limiting the requests, and for analytics.
tweet_text(String) : 140 character tweet.
user_id(String) : unique identifier of the user.
media (Object): media related to the tweet.


Response: API will provide the URL of the created tweet if a JSON response with appropriate status 

```
HTTP OK 200
{
      "user_id": "2244994945",
      "created_at": "2020-02-14T19:00:55.000Z",
      "id": "1494379925427662851",
      "tweet_text": "My First Tweet",
     "entities": {
       "media": [
          {
          "id_str": "1494379920126095365",
          "media_url": "https://pbs.twimg.com/media/FL0anqqXsAU-KDH.jpg",
          "type": "photo"
        }
      ]
    }
}

```

In case of error is appropriate, an HTTP status code will be returned.

```
GET /users/{userId}/timeline
```

Response: API will return the timeline of the user.
```
HTTP OK 200

{
	“tweets” : [
	  "1494379925427662851": {
	    "id_str": "1494379925427662851",
	    "tweet_text": "This counts as an open source contribution, right?? 😂😂😝 https://t.co/5alFRLFYvR",
	    "entities": {
	      "media": [
	        {
	          "id_str": "1494379920126095365",
	          "media_url": "https://pbs.twimg.com/media/FL0anqqXsAU-KDH.jpg",
	          "url": "https://t.co/5alFRLFYvR",
	          "display_url": "pic.twitter.com/5alFRLFYvR",
	          "expanded_url": "https://twitter.com/EddyVinckk/status/1494379925427662851/photo/1",
	          "type": "photo"
	        }
	      ]
	    },
	    "source": "<a href=\"http://twitter.com/download/iphone\" rel=\"nofollow\">Twitter for iPhone</a>",
	    "user_id_str": "633958151",
	    "retweet_count": 17,
	    "favorite_count": 236,
	    "reply_count": 15,
	    "quote_count": 5,
	    "conversation_id_str": "1494379925427662851",
	    "lang": "en"
	  }
	, .....
	]
}
```



## Database Schema
We need to store the user data, tweet data, user_follower, etc, Here is the representation of the data. 

## High-level design
As It is evident from the estimations that we need to be able to store around 
1700 tweets per second and read around 115k tweets per second.  This is a very read-heavy system. Also in case of any major events, popular celebrity tweets, there can be very spiky traffic, hence we need to scale our system accordingly.





The figure shows the basic high-level design for the Twitter service, Requests land on the load balancers which distribute the traffic to the application servers.
Media files like photos and videos are uploaded to a separate media server and its metadata will be stored with the tweet. 

At a high level, we will need the following components in our Newsfeed service:

- Web servers: receive the request for publishing the post via the REST API. The web servers check for valid authentication and authorization before allowing users to post. Besides this, there can be additional validations for example character limit, rate limiting, and anti-abuse check. 

- Tweet service: Tweets coming in are processed by the Tweet service. Tweets are written in a NoSQL DB like Cassandra.
- Media Service: It is responsible for storing the media like photos and videos associated with the tweets. Media files are uploaded separately and then linked to the tweet.
- Newsfeed generation service: It is responsible for creating the relevant news feed for all the active users. The feed will constantly keep on updating and new feed items will be reflected in the news feed of the user.
- Notification service: It is responsible to notify the users about the updates from the people the user follows.
- Analytics service: It is used to analyze tweets and generate trending items.

## Home Timeline/ Newsfeed Generation: 

Home Timeline or the newsfeed is generated from the tweets of people that the user follows. 
To generate the timeline we need to follow the below steps.
Query the social graph to get the list of users (followee), the current user follows.
Get User timelines for every followee.
Rank the tweets from this timeline based on various ranking parameters/signals in reverse chronological order.
Store the feed-in cache and return the first page of tweets.

There are two approaches to generate the news feed: 
Online Generation: We generate the news feed when the user comes online and loads the home page. 

Advantages:  Simpler to implement 

Disadvantages: This approach is not scalable as news feed generation is an expensive process. Also as new tweets constantly come in, we need a mechanism to rank and add new posts to the feed. 

We will need to constantly check for updates of all the followers even though they might not have posted any tweets.

 
Offline Generation: instead of generating the news feed online when the user logs in, we can generate it beforehand offline, cache it and serve it to the user as soon as he logs in.
We can choose to keep the 200 latest tweets for each user, allowing the user to scroll through 10 pages of tweets on his homepage.

Advantages : Fast: Users need not wait for news feed generation.
Scalable: Since the system is not loaded when the user comes online, this approach is more scalable. Also, we need not store the news feed for all the users in the cache. We can store the timelines of the users that were active in the last 15 or 30 days. As we scale our product, we can add intelligent systems that can generate the news feed by predicting the login patterns of the user.

Disadvantage : Think about the scenario where a celebrity with millions of followers tweets, this can lead to the regeneration of millions of timelines and can put a heavy load on the system.

## Fanout

If someone tweets, the timeline of all the followers is affected. Only if the users are active users.

Fanout is the process of delivering the tweet of a user to all of his followers. Two types of fanout models are:
fanout on writing (also called push model) and fanout on reading (also called pull model). Both
models have pros and cons. We explain their workflows and explore the best approach to
support our system.
 
Push Based (Fanout on write)  - When one of the followers has published a post, we can push the post to all of his followers. There is a persistent connection(Use sockets for push, at any given point there are millions of such sockets getting data from the push cluster at twitter.) between the server and the client over which feed is pushed.  There is a lot more processing on the Write ingest to figure out where to route a particular tweet.
 
Issues with this approach: As discussed in the feed generation part, for a celebrity with millions of followers tweets, it can put a heavy load on the system.
 
Pull Based (Fanout on reading) - When the user comes online, the client (web or mobile app) will pull the data after certain intervals or when the user refreshes the page. 
Issues with this approach: New data will be shown only after a pull request is issued. and some pull might result in no data.

Hybrid Approach - Use a combination of the Pull and Push based approach. Wherein we push data for the users with a limited number of followers, we can limit that too only for the currently online users. For the celebrity user, the client will pull the updates.



The above image shows a detailed overview of the feed publishing mechanism.

References
https://www.infoq.com/presentations/Twitter-Timeline-Scalability/

