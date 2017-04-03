---
title: Scaling Zappa apps using Postgres on AWS Lambda - Multiplex DB connections with PgBouncer
author: Michaël Krens
date_created: 04-02-2017
format: markdown
---

So we have a great (Django) app running on AWS Lambda with an RDS Postgres DB that in theory scales like a charm. Until it actually starts scaling... 

`OperationalError: FATAL:  remaining connection slots are reserved for non-replication superuser connections`

Oh no! Lambda might scale to thousands of containers running your app but your database is not accepting any connections anymore! Of course your Postgres should be beefy enough to handle the needed connections but you might have bursts that you also need to handle.

This can now become quite a reasonable scenario with the awesome new [Zappa Async](https://blog.zappa.io/posts/zappa-introduces-seamless-asynchronous-task-execution) functionality. Especially because when you dispatch tasks they will immediately be turned into Zappa events and handled by AWS Lambda. No queue, no delays. Creating a 1000 tasks? Lambda will scale and execute them as fast as possible and thus... exhausting DB connections. Note that this didn't happen when you were using Celery in the past as there the number of workers are fixed. We now have thousands of workers!
 
 How do we normally handle a scenario where we have more workers making DB queries than the available max client connections on your DB? Connection pools!
 
 This is exactly where we have a problem on AWS Lambda as we can't share a pool between the containers. Or can we? Yes! [PgBouncer](https://pgbouncer.github.io) to the rescue.
 
 PgBouncer can multiplex the queries of the 1000+ clients that want to connect to Postgres over a pool of connections (by default 20). PgBouncer basically acts like a proxy and keeps clients waiting for available connections to the DB (instead of failing). Downside obviously is that you'll have to run and manage an instance of PgBouncer on EC2 or ECS. Hopefully AWS will add some PgBouncer functionality to RDS in the future as they have [introduced pgbouncer-rr](https://aws.amazon.com/blogs/big-data/query-routing-and-rewrite-introducing-pgbouncer-rr-for-amazon-redshift-and-postgresql/) (quite) some time ago already.
   
Crucial configurations for PgBouncer are:

```
pool_mode=transaction  # Server is released back to pool after transaction finishes. default: session
```

```
max_client_conn=2000  # We'll need something high for all lambda containers. default: 100
```

```
default_pool_size=20  # Depending on the size of your DB you can increase this. default: 20
```

For the `max_client_conn` setting be sure to check your ulimits as when increased the file descriptor limits should also be increased. Check the [PgBouncer docs](https://pgbouncer.github.io/config.html) for more information.

The transaction pool mode is important as Lambda containers might keep sessions open while not in transactions which decreases the throughput for our use case.

To check what your current client connection limit is on your DB run `show max_connections;`. The `default_pool_size` setting should be lower.

After setting up and running a PgBouncer instance you just need to point your DB host setting to the PgBouncer instance host and port and everything is handled. Nifty!

Be aware though that by default PgBouncer runs without any authentication. Which means anyone can connect to your DB without user/pass. You'll definitely want to set that up!
 
Happy scaling!
 
