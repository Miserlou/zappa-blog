---
title: Zappa Introduces Seamless Asynchronous Task Execution
author: Rich Jones
date_created: 03-29-2017
format: markdown
---

New feature alert!

As of today's new version, `0.40.0`, [Zappa](https://github.com/Miserlou/Zappa) now offers the ability to seamlessly execute functions asyncronously in a completely seperate AWS Lambda instance, without any configuration whatsoever!

This should be useful for anybody who needs to execute long running tasks via API calls, massively execute tasks in parallel, or replace their use of Celery.

For example, if you have a Flask API for ordering a pie, you can call your `bake` function seamlessly in a completely seperate Lambda instance by using the `zappa.async.task` decorator like so:

<script src="https://gist.github.com/Miserlou/48ac23deb60cd21a74fecd5f1d558a8c.js"></script>

And that's it! Your API response will return immediately, while the `make_pie` function executes in a completely different Lambda instance.

### Task Sources

By default, this feature uses direct AWS Lambda invocation. You can instead use AWS Simple Notification Service as the task event source by using the `task_sns` decorator, like so:

<script src="https://gist.github.com/Miserlou/9a0bb76467129bf4f7819f3a85b7fa1a.js"></script>

Using SNS also requires setting the following settings in your `zappa_settings`:

<script src="https://gist.github.com/Miserlou/bd2d9447aaf3703e3f5b7e2823c70a76.js"></script>

This will automatically create and subscribe to the SNS topic the code will use when you call the `zappa schedule` command.

Using SNS will also return a message ID in case you need to track your invocations.

### Direct Invocation

You can also use this functionality without a decorator by passing your function to `zappa.async.run`, like so:

<script src="https://gist.github.com/Miserlou/7bd539df3b9eda7eb965a008bd284464.js"></script>

### Restrictions

The following restrictions to this feature apply:

* Functions must have a clean import path -- i.e. no closures, lambdas, or methods.

* `args` and `kwargs` must be JSON-serializable.

* The JSON-serialized arguments must be within the size limits for Lambda (128K) or SNS (256K) events.

All of this code is still backwards-compatible with non-Lambda environments - it simply executes in a blocking fashion and returns the result.

### Thanks!

This is an evolution of one of the earliest ideas for Zappa, dating all the way back to [issue #61](https://github.com/Miserlou/Zappa/issues/61), with new life breathed into it with [issue #603](https://github.com/Miserlou/Zappa/issues/603).

It's very fresh code, so please use it at your own risk for now, and please [file bug reports](https://github.com/Miserlou/Zappa) and pull requests if you find room for improvement. We have already identified some performance improvements which will likely land in the next version.

I'd like to thank Michaël Krens, Rich Fernandez, Alex Ehlke, Matthew Crowson and the others in Slack and on GitHub for their contributions to the discussion, and to especially thank Nam Ngo and Schuyler Duveen for their contributions to the code.
