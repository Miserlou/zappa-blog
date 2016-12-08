---
title: Server-less Framework Comparison - Zappa Versus Chalice
author: Rich Jones
date_created: 11-17-2016
format: markdown
---

_On July 11, 2016, 6 months after the first published version of [Zappa](https://github.com/Miserlou/Zappa), Amazon announced the preview release of [Chalice, now the Python Severless Microframework for AWS](https://aws.amazon.com/blogs/developer/preview-the-python-serverless-microframework-for-aws/). This was a fairly big shock to our community at the time, as it looked as though our Free and Open Source project was being undercut by Amazon's propietary offering. Not only that, but it felt like their interface and even their presentation of the product was a̶ ̶d̶i̶r̶e̶c̶t̶ ̶r̶i̶p̶-̶o̶f̶f̶ ̶o̶f̶ inspired by our efforts, potentially designed to ~~~trick~~~ help consumers and lock them into the AWS ecosystem._

I don't want to attribute any direct malice there, (this is probably just a case of a good idea being executed in multiple ways), _but_ Amazon does have a fairly long history of building their own proprietary or locked-in versions of software products and services built on top of AWS infrastructure.

So, now that both products have matured even further, I think it's time that we do a comparison of the two frameworks. Obviously, as one of the authors of Zappa I'm strongly biased, so take my opinions for what they are, but it is my strong belief that although Chalice has some interesting features, *Zappa is the vastly superior product* for these important reasons:

* Zappa **does not lock you in** to the AWS ecosystem.
* Zappa has vastly **more useful features**.
* Zappa is **battle-tested**, used in production by medical, scientific and banking users.

### Against Vendor Lock-In

This is the big headline here: Chalice locks you into using AWS services - Zappa doesn't.

With Zappa, you can run existing Python web applications on AWS Lambda thanks to the magic of WSGI. More importantly, you can _leave_ AWS at any time, meaning you can run the same application on any web server your like, be it VPS, on Heroku, or on your own bare metal. (You will lose some of the event-driven benefits which AWS can provide, but you will always have the option to use other replacements if that's what you decide.)

With Chalice, you will have to re-write your application to use the Chalice framework, and after that it will only ever be able to be run on AWS infrastructure.

Although AWS is still the absolutely leader in the server-less space, other vendors are catching up rapidly, and we intend to have Zappa deploy applications across multiple cloud providers to provide the most featureful, fast and resiliant applications we can.

Specifically, we are interested in adding support for IBM's OpenWhisk, a Free and Open Source server-less infrastructure package which can run in the cloud and on existing hardware. (More on that in an upcoming blog post.)

### Exclusive Features

In addition to the ability to use existing mature Python web frameworks like Flask and Django without any vendor lock-in, Zappa also sports:

* Automatic support for all AWS event sources, allowing you to build robust "hybrid" applications
* Automatic support for _hundreds_ of pre-compiled and pre-optimized C-extentions packages (SciPy, etc.)
* Support for applications which use multiple cookies simultaneously (i.e., anything with a login)
* Free, auto-renewing SSL certificates from Let's Encrypt
* CI integration
* Automatic support for "global" (multi-region) applications
* HTTP logging in the Common Log Format
* Automatic "keep-warm"
* Intelligent application and variable caching
* Zero-effort CORS
* Automatic and intelligent project initializion (`zappa init`)
* Multiple environment variable sources (local, S3)
* Multiple "secure endpoint" authentication methods (API Key, IAM, custom authorizers)
* Content-Type aliases
* Highly customizable configurations
* Managed IAM credientials with the option to supply custom credentials
* Package optimization
* VPC-awareness
* Ability to deploy HTTP-less, event-driven applications
* Custom error-reporting (ex, to Sentry or Raygun)
* Remote command invocation (include raw Python)

And many more, with many more on the way!

#### Hybrid Applications

One of the most useful of these features is the ability to build a "hybrid" application - an application which uses both traditional HTTP/API end points as well as AWS event sources.

For example, imagine that you're using Zappa to run a web forum where users can have avatars. You want your users to be able to upload any image to S3 and have a thumbnailed version of that image, but you don't want the thumbnailing to happen on a slow-seeming HTTP POST.

With Zappa, it's trivially easy to create this with a few lines of configuration:

    # zappa_settings.yml
    ---
    dev:
      app_function: your_project.main.app
      events:
      - function: your_project.users.thumbnailer
        event_source:
          arn: arn:aws:s3:::upload-bucket
          events:
          - s3:ObjectCreated:*

Then, in your existing project:

    # your_project/users.py
    import Pillow

    def thumbnailer(event, context):
        """ Upon PUT, thumbnail! """

        # Get the bytes from S3
        in_bucket = event['Records']['s3']['bucket']['name']
        key = event['Records']['s3']['object']['key']
        image_bytes = s3_client.download_file(in_bucket, key, '/tmp/' + key).read()

        # Thumbnail it
        size = (250, 250)
        thumb = ImageOps.fit(image_bytes, size, Image.ANTIALIAS)

        # Put it back on S3
        s3_client.put_object(
            ACL='public-read',
            Body=thumb,
            Key=key + 'thumbnail.jpg',
            Bucket='avatar-bucket')

Now you have a non-blocking way to perform a long-running tasks without managing any infrastructure, without affecting the performance of any of your other users, and without even leaving your project's context.

This applies to any type of AWS event source you can think of - notifications, objects, queues, streams, and even database states!

### Battle Testing

I'm very pleased to say that Zappa is used in production by companies all over the world, including banks, medical instituions, and Fortune 100 companies - including AWS themselves (from what we've been told 😉)!

I'm proud to say many of the features listed above came from Zappa users who contributed them upstream. Zappa has the features that you need because somebody else needed them, built them, and then contributed them back to the community. Zappa has over 50 contributors already, with more contributing every day. I'm also quite proud of the quality and maintainability of the code-base itself!

We've heard reports of some Zappa deployments serving over 15,000 simultaneous connections per second, per-region without any signs of trouble. We've seen Zappa used in production as part of medical and financial systems. We're even beginning to see start-up companies skip traditional deployments all together and launch their first products directly with Zappa!

In short, Zappa is ready for _serious_ work.

### Concessions

The biggest advantage that Chalice has over Zappa is related to IAM roles. Chalice has a very cool feature where it will actually inspect your Chalice application for AWS service calls and generate an IAM policy based only upon what the application needs. (I personally think that it's really quite neat!)

However, this does not seem to be a serious requirement in production. We generally see Zappa users fall into two camps:

* Individuals, who want everything to "just work" out of the box (this our default)
* Corporate users who work under very locked-down policies, who aren't allowed to create their own policies, and must use hand-reviewed and maintained policies and credentials (these users can simply provide their own `role_name`, `assume` and `attach` policies.)

Since this seems to cover the majority of users, we haven't had any requests for better automatic IAM policy generations.

That being said, I will admit that our IAM management needs work, and though I don't think Chalice's solution is how we will solve the same problem, we do intend to address and greatly improve our IAM management in the coming weeks and months.

### Conclusion

In conclusion, I'd encourage you to assess both projects on their own merits and make the decision for yourself.

Hopefully, I've convinced you that using Zappa is the right decision if you want to **avoid vendor lock-in**, have access to **lots of useful features**, and use **battle-tested** software.

Like what you see? Give us a star on [Zappa's GitHub page](https://github.com/Miserlou/Zappa)!

Still got questions? Join us on [Slack](https://slack.zappa.io)!

Thanks!,

Rich