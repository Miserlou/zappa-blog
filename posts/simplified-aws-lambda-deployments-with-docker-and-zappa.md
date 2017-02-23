---
title: Simplified AWS Lambda Deployments with Docker and Zappa
author: Matthew Crowson
date_created: 02-21-2017
format: markdown
---

Updating your project hosted on AWS Lambda via Zappa is as easy as calling 
`zappa update`. Zappa takes the Python packages installed into your virtual 
environment and sends them and your project up to Lambda. Boom! Your code 
is now updated. 
 
### The problem with environments
The Lambda working environment and operating system is often different 
than your developer working environment. So the Python packages in your 
virtual environment might be built perfectly for your local needs on 
MacBook or your super sweet Windows XP desktop. Zappa will try to automatically
use pre-compiled [lambda-packages](https://github.com/Miserlou/lambda-packages) or
"manylinux" wheels on pip, but that doesn't yet cover all packages you might need,
especially if you're customizing them.

### Enter Docker
The blessed souls over at LambCI have released a [Docker image](https://github.com/lambci/docker-lambda) 
that looks, smells and sounds like the AWS Lambda environment. This 
means we can `pip install` to our heart's content and know that the packages 
will run in Lambda. The only sad part is that this Lambda environment 
is pretty bare bones (just like the real lambda) and might not have 
all of the OS packages needed for us to even install the Python packages.

### [@danielwhatmuff](https://github.com/danielwhatmuff) Saves the Day
Building off of LambCI's Docker image, Daniel Whatmuff has constructed 
a new [Dockerfile](https://github.com/danielwhatmuff/zappa) for us 
that installs some of these missing dependencies. All that was left 
was to build the image and put it on [Docker Hub](https://hub.docker.com/r/mcrowson/zappa-builder/) 
for reuse.

### One Line Deployment 
With the docker image ready to go, we can call it from our project's working 
directory to update our Zappa project. 

```bash
docker run -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_DEFAULT_REGION=us-east-1 -v $(pwd):/var/task --rm mcrowson/zappa-builder bash -c "virtualenv docker_env && source docker_env/bin/activate && pip install -r requirements.txt && zappa update dev && rm -rf docker_env"
```

We do a few things when we call this command. 
 - Environment variables are passed for AWS so Docker can call update with the appropriate permissions.
 - The current directory is attatched as a volume in Docker's working environment.
 - Lastly, the bash commands to build a virtual environment, install requirements, update zappa, and cleanup.
 
Your project is now updated and the updated packages are Lambda compatible! 
If you're environment is still unable to pip install certain Python packages, 
you may need to build your own Docker image similar to how [@danielwhatmuff]([Dockerfile](https://github.com/danielwhatmuff/zappa))
built the one we are working with here, but inclusive of your missing system depdendencies. 
