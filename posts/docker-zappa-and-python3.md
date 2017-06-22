---
title: Zappa, Docker, and Python 3 - Updated guide to fixing your environment
author: Edgar Roman
date_created: 06-16-2017
format: markdown
---

In a [previous post](https://blog.zappa.io/posts/simplified-aws-lambda-deployments-with-docker-and-zappa),
Matthew Crowson reviewed how to get a docker environment that closely resembles the AWS
lambda environment.  This allows you to have a local environment that can be used to debug
package and other system library installations.  

Since that blog post was published, AWS announced support for Python 3.6 and some things have changed.  This post goes through the process of setting up your own docker environment for zappa using Python 3.6.

### Pull the Docker Image

We'll pull directly from the [lambci](https://github.com/lambci/docker-lambda) images directly by running:
`docker pull lambci/lambda:build-python3.6`

### Setup your Dockerfile

The Dockerfile is important to allow you to customize any system library dependencies.

<script src="https://gist.github.com/edgarroman/52f643cdc5482a0bd88684c5f7b7c419.js"></script>

If you need to compile stuff or install any packages that are *not* in [lambda-packages](https://github.com/Miserlou/lambda-packages) you can install them using Dockerfile

### Build your Docker Image

Easy by running `docker build -t myzappa .`

### Add an alias for quick access

<script src="https://gist.github.com/edgarroman/33092e113416f668f44649910f35551c.js"></script>

Note there are many [alternative options](https://edgarroman.github.io/zappa-django-guide/setup/#inital-setup) depending on how you have setup your [AWS credentials](https://edgarroman.github.io/zappa-django-guide/aws_credentials/#setup-local-account-credentials)

### Get rolling

Now you can easily start up the docker container by:

```
cd /myzappaproject
zappashell
```

And setup your virtual environment

```
python -m venv ve
source ve/bin/activate
pip install -r requirements.txt
```

NOTE: It is very important that you create, activate, and install packages into the virtualenv only in the docker shell. This will prevent any incompatibilities with the local system environment and the docker environment.

### More info and advanced setup

More info including a dual-python version setup can be found [here](https://edgarroman.github.io/zappa-django-guide/setup/)