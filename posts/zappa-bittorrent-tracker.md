---
title: Announcing zappa-bittorrent-tracker
author: Rich Jones
date_created: 09-26-2016
format: markdown
---

I'm pleased to announce one of the most interesting use cases of Zappa yet: **[zappa-bittorrent-tracker](https://github.com/Miserlou/zappa-bittorrent-tracker)**!

**ZBT** is the world's first completely server-less* tracker. It can scale to handle millions of peers out of the box, requires no permanent infrastrcture, and it has a choice of two interesting data-stores powering it: **S3** and **DynamoDB**.

It's a proof-of-concept more than anything else at the moment, but it could be of use to anybody who needs to distribute data via torrents to millions of peers without any ops burden.

*_Server-less, in this case, means without any-permanent infrastructure, using AWS Lambda and API Gateway. This project has nothing to do with the BitTorrent DHT._

### Server-less, Database-less

As an experiment, I thought it'd be interesting to have the option to use Amazon's S3 service as a data-store rather than a traditional database. Normally S3 is just used for receiving and serving static binary assets, but because it has incredibly low latencies for connections to other AWS resources (like our AWS Lambda) and can handle essentially unlimited parallel connections, it actually works pretty well as a generic key-value store for our purposes here. I haven't tested it extensively, but it's possible there are potential race-conditions in using it in this fashion, however I don't think that'll be a problem for the average torrent use case.

As an alternative, ZBT can also use Amazon's DynamoDB as a database. This is a much more traditional key-value store, which seems to have pretty excellent performance and scalability on its own, however it's much more expensive.

### Scalability

With the training wheels taken off your AWS account, you should be able to handle 5,000 simultaneous connections per second, so with a 30-minute announce interval, this set-up should be able to handle 9,000,000 peers out of the box. With a multi-region deployment and a larger announce window, this should be able to scale to 100,000,000+ peers without much difficulty. Awesome!

### Future

I'm mostly just using this for my own testing purposes while I work on a new BT client, so we'll see what happens in the future.

Obviously, it's not a great solution to anybody wanting to host a pirate BitTorrent tracker, as Amazon will be subject to government censorship, but I think ZBT could be really useful for some other use cases, such as distributing Linux ISOs and packages, or for delivering large amounts of game data, free sample libraries, stuff like that.

If you end up using ZBT for anything, please let me know! PRs are, of course, gladly accepted.

Enjoy!,

Rich