---
id: notification-system
title: Design Notification System
sidebar_label: Design Notification System
---

A notification system is a system to send notification alerts to the users. Notification Alert is a piece of information used to provide some updates to the user for example - updates about the ongoing order, new offering, meeting reminder, bill due alert, OTP notification, etc.

Notification systems are an integral part of any mobile or web application and act as a channel to communicate with users. Notification can be of multiple types, Mobile push notification, Email Notification, SMS, Whatsapp chat.  Let us call them different notification channels
 

## Functional Requirements 

Notification Types: Push(Desktop, IOS, and Android), SMS, and email.

Notification Preferences: The system should respect User preferences or settings. For example, a user who has opted out of a particular type of notification should no longer receive the notifications.

Notification Scale: Our notification system should be able to handle a scale of close to 10 Million notifications per day.

Notification Prioritization - certain messages like OTP are higher priority, whereas promotional messages are lower priority

## Non Functional Requirements

Always Available: Since we are using the Notification System to send critical information to the customer, the system should be always available.

Latency: Notifications should be delivered in almost real-time. Slight delays are acceptable when the system is under peak workload.

Pluggable: Our notification system should allow additional channels without much effort and redesign.

Scalable: the system should be scalable to cater to increased loads without much increase in latency.
 
Client Services: The services on the left represent different services that want to send notification messages via our Notification service. It can be an auth service, trying to send OTP over SMS or email channel, or it could be an order service trying to send order updates.
Application Servers: expose the API‘s which client services can use to initiate notifications
Validate incoming messages for valid emails, phone numbers, etc.
Verify user settings if the user has opted out of such notifications.
Fetch other data required to send the notification for example notification templates, device info, etc.
Push message to the appropriate message queue for processing.
Cache: used for caching the user settings, device info, and notification templates to prevent frequent DB trips.
DB: to get info in case of a cache miss.

Message queues: Asynchronous way of handling the messages, messages are buffered in the queue till they are picked by the workers for processing. Different queues are used for different channels like email, SMS, etc.

Workers: the set of servers that pull messages from the messaging queue and process them one by one by sending them to corresponding services.  Here are few examples
- SMS
- Email
- Mobile Push Notifications
  a) iOS b) Android



### API

POST https://notification-api.example.com/api/v1/sms

Request body
```
{
	"user_ids" : [
			"ee0ad2aa-2f71-11ec-8d3d-0242ac130003", 
			"980ea932-76ee-4816-b46e-4c26d99f0f9a",
			"5cea9e39-f2f1-4277-aeaf-5eb344f4a7bd"
		    ],
	"sub": "The Big Festival sale starts in 10 minutes!",
	"body": {
		"type": "text/plain",
		"value": "Don't miss the latest deals, visit abcd.com now!"
	}

}
```

### Reliability
Our notification system should be reliable so that the customer does not miss any update meant for him/her. Even in the case of peak traffic, we should avoid any data loss. Some amount of data delay is acceptable though. For this, we can employ acknowledgment and delivery semantics by acknowledging the message only in case the processing of the message was successful at the worker end. We can also keep an append-only log of all notifications for auditing purposes at the worker level. 


### Analytics
Analytics is an important component for notification systems as it will help us determine how effective our system is by collecting engagement statistics like open rate and click rate.
This will provide great feedback about the customer needs and help us understand the customer behavior effectively. 
