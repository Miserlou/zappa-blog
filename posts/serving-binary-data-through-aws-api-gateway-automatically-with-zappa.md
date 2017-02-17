---
title: Serving Binary Data Through AWS API Gateway Automatically with Zappa
author: Rich Jones
date_created: 02-17-2017
format: markdown
---

Version 0.36.0 of Zappa just got published to PyPI with a bunch of great improvements, but here's my favorite: Automatic support for serving binary data files!

Previously, Zappa users could only serve plaintext responses through the API Gateway, or they would have to manually Base64 encoded any binary data, or they would have to send files to S3 to be served. Now, because Amazon has added native binary/Base64 support, you don't have to do this manually anymore!

Zappa actually takes this a step further and processes your response at the middlware layer by checking the MIME-type of your application's response and properly configurating the response to the API Gateway so that your data gets served correctly both for binary and plaintext responses without any configuration on your part.

This should make is much easier to deploy apps that serve dynamic binary content, such as image manipulation programs or PDF document generators.

As a simple example, here's how to serve a static file using Flask:

<script src="https://gist.github.com/Miserlou/fcf0e9410364d98a853cb7ff42efd35a.js"></script>

And as a slightly more complex example, [here's a Flask/Pillow/OpenCV application](https://github.com/wobeng/zappa_resize_image_on_fly) that can dynamically resize images.

Big thanks to [Welby Obeng](https://github.com/wobeng) for implementing this feature!

Enjoy!,

Rich