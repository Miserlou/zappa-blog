---
title: Large Applications on AWS Lambda 
author: Matthew Crowson
date_created: 02-01-2017
format: markdown
---


Over the past year, Zappa users has been creating and migrating web applications to AWS API Gateway / Lambda by using Zappa. The limitations on the projects that we can launch on Lambda are [limited](http://docs.aws.amazon.com/lambda/latest/dg/limits.html) by AWS however. The major limitation that we have run into is when the packaged application, with all its dependencies, hits the 50M zip limit for the lambda handler. 

We have seen this in the community as people try to [finagle large libraries like scikit-learn](https://serverlesscode.com/post/deploy-scikitlearn-on-lamba/) into a lambda function. It is not easy to get everything under 50M when trying to leverage certain libraries. 

### Slim Handler
Zappa deployments now support up to 500M of zipped up Python projects. Simply set `“slim_handler”: true` in zappa_settings.json and your large projects can now serve up requests from Lambda without a server. 

### But How?
This is accomplished in two ways. First, Zappa zips up the large application and sends the project zip file up to S3. Second, Zappa creates a very minimal slim handler that just contains Zappa and its dependencies and sends that to Lambda. 

When the slim handler is called on a cold start, it downloads the large project zip from S3 and unzips it in Lambda’s shared /tmp space. All subsequent calls to that warm Lambda share the /tmp space and have access to the project files; so it is possible for the file to only download once if the Lambda stays warm.


