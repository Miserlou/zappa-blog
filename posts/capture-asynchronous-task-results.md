---
title: Capture Asynchronous Task Results
author: Sean Coates
date_created: 09-21-2017
format: markdown
---

Nearly six months ago, we [posted](https://blog.zappa.io/posts/zappa-introduces-seamless-asynchronous-task-execution) about a new feature in Zappa: Asynchronous Task Execution.

With this week's release of `0.44`, [Zappa](https://github.com/Miserlou/Zappa) now offers the ability to capture the output from these asynchronous tasks, with very little extra configuration.

In that introduction post, we told you "this should be useful for anybody who needs to execute long running tasks via API calls, massively execute tasks in parallel", but back then there was no way to fetch the *results* of those asynchronous tasks.

Now there is.

Let's get back to baking:

<script src="https://gist.github.com/scoates/027488c39d5407bbfa125ff3f3a9a622.js"></script>

With this example, we can bake a large number of things in a short amount of time. Each of these things takes 1 to 10 seconds to bake, but that doesn't mean that our request takes up to 10 seconds per item.

Consider this sample run:

<script src="https://gist.github.com/scoates/7a8639ccc014a35f5646f1e0434d1d7c.js"></script>

We did 67s worth of baking in less than 11 seconds, and we even found out what we baked.


### Zappa Settings

To use this functionality, you'll need to configure `zappa_settings.json` with a few new values:

<script src="https://gist.github.com/scoates/cb9058046cea80415ae3ddbc55c793a2.js"></script>

This lets Zappa create and use a DynamoDB table, and set up its capacity.

### Thanks!

As with the earlier post about asynchronous jobs, result capturing is very fresh code, so please use it at your own risk for now, and please [file bug reports](https://github.com/Miserlou/Zappa) and pull requests if you find room for improvement.
