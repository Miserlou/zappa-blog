---
title: Use SQLite with Django On AWS Lambda with Zappa
author: Rich Jones
date_created: 02-07-2017
format: markdown
---

Here's a quick tip on how to use your local development SQLite database on your development AWS Lambda deployment using Zappa!

Start a new file called `dev_settings.py` in the same directory where you keep your project's base settings and put this in it:

<script src="https://gist.github.com/Miserlou/5a98aeacda10662f4af1e0b8050fb244.js"></script>

This will copy your local development SQLite database to Lambda's temporary working space when your application is first loaded into the cache.

Then, just update your `zappa_settings.json` accordingly by setting `django_settings` to `your_app.dev_settings`.

Finally, run `zappa update` and you're done!

You'll find that it's actually pretty snappy and changes persist for at least a little while, but obviously you'd never want to use this for any production data.

Enjoy!

Rich
