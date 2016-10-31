# Continuous Zappa Deployments with Travis

First of all, what is a Continuous Deployment? To make things short, it's basically just a fancy term telling you to push your code quickly, as soon as it's ready.

If you are using Zappa, you are probably doing this already. All you have to do is to run the `update` command and Zappa sets everything up for you. In this small article, we are going to automate the `zappa update`-step by telling Travis to do it for us. 

This can be useful in many different ways. For instance, if you are working on a project with a team and don't want to share the AWS credentials with every developer, or if you don't want to wait for your tests to finish before deploying your code.

At [pyup.io](https://pyup.io) I'm using Zappa a lot for internal micro services. With a growing list of services, updating each individual service became unmanageable real quick.

## Writing a small test app

We are going to start with a super simple Zappa deployment running flask.

Create a new project directory, a fresh virtualenv and activate it. If you are using virtualenvwrapper like me, this might look like this:

```
mkdir zappa-cd && cd zappa-cd
mkvirtualenv zappa-cd-app
```

Next, we create a `requirements.txt` file and install it:

```
echo "zappa==0.28.0\nflask==0.11.1" > requirements.txt
pip install -r requirements.txt
```

Create a new file `app.py` with the following content:

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return "hi from zappa!"
```

Looking good so far!

We now need to intialize the project. Simply run `zappa init` and use the defaults:

```
zappa init
[snip]
```

Open the newly generated `zappa_settings.json` and add the `aws_region` and the `project_name` manually:

```
{
    "dev": {
        "app_function": "app.app",
        "s3_bucket": "zappa-f55fa750i",
        "aws_region": "us-east-1",
        "project_name": "zappa-cd",
    }
}
```

We need to do this because the Travis environment we are going to set up in a moment might be a bit different than what we are using locally.

Our app is now ready to be deployed for the first time, run:

```
zappa deploy dev
```

Once the deployment is complete, visit the url zappa presents to make sure everything is working:

```
open https://whatever.execute-api.some-region.amazonaws.com/dev
```

Great! But really, this is nothing special so far. We are going to add a automated Continous Deployment with Travis next.

## Adding Continous Deployments with Travis

If you haven't done this already, create a new repo on GitHub, push your code and add the repo to [Travis](https://travis-ci.org).

In order for Travis to pick up our repo correctly, we need to add a `.travis.yml`:

```
language: python
python: '2.7'
script: echo "all tests passing"
after_success:
 - pip install -r requirements.txt
 - zappa update dev
```

This should be familiar if you have used Travis before. It basically just sets up a testing environment for Python 2.7 and runs a fake test.

The part that makes this a continous deployment is in `after_success`. This tells Travis to run `zappa update dev` whenever our tests are passing. Extremely simple.

The last thing to do is to add our AWS credentials to Travis. There are multiple ways to do this, but the easiest is the web interface.

Click on the *Settings* tab and add your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. That's it!

![aws credentials](https://raw.githubusercontent.com/Miserlou/zappa-blog/master/images/travis_aws_credentials.png)

Now all we have to do is to add `.travis.yml` to git and to push the file. This will trigger a new build for us. Once it has finished, check out the build log at the end. 

It should tell you that "Your updated Zappa deployment is live!". 

![travis deployment](https://raw.githubusercontent.com/Miserlou/zappa-blog/master/images/travis_deployment.png)

Awesome.
