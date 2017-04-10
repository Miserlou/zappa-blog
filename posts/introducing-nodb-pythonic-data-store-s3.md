---
title: Introducing NoDB - a Pythonic Object Store for S3
author: Rich Jones
date_created: 04-10-2017
format: markdown
---

<img src="http://i.imgur.com/ZymFZd8.jpg" width="400"/>

New project!

I'm happy to announce the first public release of **[NoDB](https://github.com/Miserlou/NoDB)**, an incredibly simple, Pythonic object store based on Amazon's S3 static file storage. NoDB isn't a database.. _but it sort of looks like one!_

It _sort_ _of_ does for databases what **Zappa** did for web servers. That's a bit of a stretch, but it's a step in that direction.

It's mostly useful for **prototyping**, **casual hacking**, and (maybe) even low-traffic **server-less databases** for **[Zappa](https://github.com/Miserlou/Zappa) apps**!

### NoDB In Action

**NoDB** is super easy to use! You simply make a NoDB object, point it to your bucket and tell it what field you want to index on. After that, you can save and load literally anything you want, whenever you want! Magic!

<script src="https://gist.github.com/Miserlou/8fa47edb90542c18cb2dc59c559b8dd9.js"></script>

By default, you can save and load any Python object.

Here's the same example, but with a class. Note the import and configuration is the same!

<script src="https://gist.github.com/Miserlou/4991ecaacad899d2c075315a85f665fc.js"></script>

You should read the [README file](https://github.com/Miserlou/NoDB) for more information about how to use different serializers and other more advanced features.

### But.. why?

Not everything in the world needs a full on relational database. Amazon previously provided a service for those scenarios, SimpleDB, but it seems that they've abondonded it, and they never really provided a natural, Pythonic interface to it anyway.

I can see a few use cases for **NoDB**:

* Prototyping schemas
* Storing API event responses for later replay
* Capturing event logs
* Storing non-relational analytics data
* Firing Lambda event triggers
* Version controlling evolving Python objects

Mostly, I'm using **NoDB** to prototype a microservice that I don't know what the final schema will be yet. Once I've built that out fully, it should be easy to switch over to something with a schema built on top of Capless' [K.E.V.](https://github.com/capless/kev), which adds a schema and relational layer on top of this server-less database philosophy.

Another interesting thing about **NoDB** is that you can use the S3 object writes as an event trigger for Lambda events, so your **NoDB** writes can be triggers for Zappa functions. (You can also do this for DynamoDB, but not for SimpleDB.)

### In "Production"

In small-scale production, **NoDB** should work fairly well for anything up to about 100 simultaneous connections per second with spikes up to 800, as per AWS's S3 limitations documentation.

For applications with sparse, non-relational data, this may be totally satisfactory for your purposes. For instance, for my [zappa-bittorrent-tracker](https://github.com/Miserlou/zappa-bittorrent-tracker), this would probably be a perfectly acceptable solution.

That being said, I wrote this yesterday, so I wouldn't recommend using it for anything "real" yet unless you really know what does under the hood. Still, I hope it can be helpful for speeding up your development times.

### Future Work

This was just a Sunday project, but if it's useful for people, there are plenty of features that could be added to make it even more useful.

For instance, compression of the data on S3, and using time-scheduled or write-event triggered Zappa functions to perform asyncronous indexing for faster lookups.

Anyway, I'd love to know what you guys think about it, especially if you actually try to take it for a spin. It should hopefully speed up your prototypes.

Enjoy!

Rich
