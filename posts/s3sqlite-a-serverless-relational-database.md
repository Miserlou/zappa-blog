---
title: S3SQLite - A Serverless Relational Database 
author: Rich Jones
date_created: 10-17-2017
format: markdown
---

New feature alert!

As a spiritual descendant of our [NoDB](https://github.com/Miserlou/NoDB) project, I've written a new server-less database engine for Django: [s3sqlite](https://github.com/Miserlou/zappa-django-utils).

It's a thin wrapper around the normal Django SQLite database engine, but it automatically syncs the DB file with S3.

Obviously, this will cause problems for high-write applications, but for high-read applications without concurrent writes, it scales very well, it's trivial to set up, and it's orders of magnitudes cheaper that AWS RDS.

To use it, first install [zappa-django-utils](https://github.com/Miserlou/zappa-django-utils) via `pip`:
    
    $ pip install zappa-django-utils

Add to your installed apps:

    INSTALLED_APPS += ('zappa_django_utils',)

Then, in your Django project's `settings.py` file, add the following:

<script src="https://gist.github.com/Miserlou/8a17a9551ebc289c74137b0c78619c0c.js"></script>

And.. that's it!

This is a pretty quick hack, but I have some ideas about ways that can significantly improve this that I hope to implement in the not-too-distant future, so stay tuned for that.

Enjoy!

Rich
