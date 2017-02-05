---
layout: post
title: MongoDB Query Examples
tags: [MongoDB]
categories: [MongoDB]
image:
  background: triangular.png
---

In this blog I will focus on explain how to perform common operations on a given collection in MongoDB.

Let consider a scenario where we have a messaging APP. An user can communicate with one or more users through the application and the system will
track all the communication in MongoDB.

<figure class="half center">
	<img src="/images/mongo/messaging.png" height="800px"></img>
</figure>

Let consider a scenario where we have a messaging APP. An user can communicate with one or more users through the application and the system will
track all the communication in MongoDB.

<figure class="half center">
	<img src="/images/mongo/messaging-design.png" height="800px"></img>
</figure>


We have a collection named "messages" in MongoDB. 

Below is a sample documents that are stored in the collection:



{% highlight yaml %}

{
	"_id": "582a15149bbc2d2f8898628b",
	"senderUserId": "1234",
	"receiverUserId": "7890",
	"message": "Hi, How are you?",
	"sentDate": ISODate("2016-11-14T19:48:36.379Z")
},
{
	"_id": "582a15149bbc2d2f8898628b",
	"senderUserId": "7890",
	"receiverUserId": "1234",
	"message": "I am doing well. How about you?"
	"sentDate": ISODate("2016-11-15T2:10:11.123Z")
}, {
	"_id": "582a15149bbc2d2f8898628b",
	"senderUserId": "5678",
	"receiverUserId": "2222",
	"message": "Hi, How are you?"
	"sentDate": ISODate("2016-11-17T01:04:6.079Z")
}

{% endhighlight %}

 



