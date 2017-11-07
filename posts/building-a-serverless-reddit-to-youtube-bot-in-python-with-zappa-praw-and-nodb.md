---
title: Building a Serverless Reddit to YouTube Bot In Python With Zappa, PRAW, and NoDB
author: Rich Jones
date_created: 11-07-2017
format: markdown
---

_tl; dr: I got frustrated, made a bot, deployed a bot, lost a bot, and now I'm going to share it with you!_

Reddit.com recently switched to preferring their own home-made video player for movies and long gifs. I'm not sure what's going on under the hood, maybe they need to switch to a better CDN or a more versatile JavaScript player, but I find this player to be vastly inferior in terms of performance and usability to other web video players, especially on mobile.

So, in a momement of frustration, I wrote a bot which will automatically upload all Reddit video player submisisons to YouTube.

To do this, I used the awesome Python library `praw`, as well as `NoDB` for storing data, and, of course, `Zappa` for deployment. In addition, I use `youtube-dl` and `youtube-upload` for downloading and uploading the videos, which are powered by `ffmpeg` under the hood.

This is a little bit janky, hour-long hack bot, but hopefully it'll help you to see how a Zappa bot gets built.

## Installation

In a new project directory and virtualenvironment, `pip install praw nodb zappa youtube-dl google-api-python-client progressbar2`. `youtube-upload` isn't available on `pip`, so you'll need to `git clone https://github.com/tokland/youtube-upload` into your project directory. Next, grab the `x86_64` static `ffmpeg` build John Van Sickel's [static ffmpeg builds page](https://johnvansickle.com/ffmpeg/) and extract it into your project directory.

## Credentials

Next, you'll need to create a new Reddit account for the bot to post under. Then, go to preferences and create a new application. Make sure you create a `script` application, rather than a `web` application.

Now, do the same for the YouTube account. Make a new "channel", then get an API keypair and authorize YouTube access for that keypair. Because of how YouTube uploads work, you'll now need to upload a test video manually with the `youtube-upload` executable in the `bin` folder. This will give you a link to visit in your browser to confirm your account, and will generate some credentials file in your home directory. Move this credentials file to your project directory.

You'll also need to make a new private S3 bucket to store your `NoDB` information.

## Reading a Stream

Now, create a `bot.py`. First, our imports and our NoDB set up:

<script src="https://gist.github.com/Miserlou/e9812c3b40de88b194c6dd8dc803be07.js"></script>

Next, make a main method and use `praw` to read the stream of new posts to the reddit video domain.

For each of the items in the stream, check the NoDB to see if we've already processed this item, and if not, fire off the item processor.

<script src="https://gist.github.com/Miserlou/11c63d4cceca47c4df644f30541cd01c.js"></script>

## Downloading

Now that we've got the submission we want to mirror, let's download it!

First, we want to make sure we don't accidentally upload any pornography to YouTube, so we'll remove all NSFW posts and a few NSFW subs.

After that, we'll get the HLS stream from the submission item and call `youtube-dl` on it.

There are a few important things to notice here. First, note that we are being lazy and calling another python executable directly. On Lambda, this is located at `/usr/bin/python2.7`. Next, we are downloading to `/tmp`, since this is the only writable directory on Lambda. Finally, we are passing the location of our own `ffmpeg` build for `youtube-dl` to use.

<script src="https://gist.github.com/Miserlou/da7b7534cf729a488b93337dd6184f57.js"></script>

## Uploading

Now that we've downloaded our file, we need to upload it to YouTube. We'll use a similar process of calling the `python` executable directory, but with a twist to get around some of funkiness of Google's client library.

<script src="https://gist.github.com/Miserlou/65928bbd05b79f8a34e456e3db6739dd.js"></script>

Notice that we supplying the locations of our `client-secrets` and `credentials-file` from our local directory, and, most imporantly, defining a custom `PYTHONPATH` for the `env` of this execution. Finally, we get the `video_id` of the result.

## Posting A Reply

Finally, we create a little Markdown comment with a link to our new video, and we post it as a reply to the video submisison:

<script src="https://gist.github.com/Miserlou/d627b9361d928fc5cdd37dda7254b748.js"></script>

## Deploying

Now, we're ready to deploy! First, type `zappa init` and point it to your source bucket and region. Since we don't need an API endpoint for this project, we can set `apigateway_enabled` to `false` in our settings.

Next, we'll want to set up a scheduled event for our function, so we create the following item for our settings file:

<script src="https://gist.github.com/Miserlou/f55ef99f5682f450725ff5f8d652a02e.js"></script>

Now, just `zappa deploy`, and you're up and running!

## Debugging

Figuring some of this stuff out took a few tries (especially the weird stuff with setting the `PYTHONPATH`.) To help debug this, I used some handy `zappa` features: `zappa tail`, which tails the Zappa output logs, `zappa invoke`, which directly invokes a function, and `zappa invoke --raw`, which directly executes supplied Python code in the remote environment.

Using `zappa invoke --raw` allowed me to invoke testing code without uploading the whole new package every time, which saved a bunch of time!

## Banned :(

Response to the bot is mixed so far, some people seem to [really love it](https://www.reddit.com/r/gifs/comments/7baksp/in_russia_5_people_dont_jump_you_1_person_jumps_5/dpgj3uc/), but some people seem to hate it too.

Unfortunately, the bot immediately uploaded some NSFW content that made it past the filter and got a content strike from YouTube. Shortly after that, it got banned from the Aquariums subreddit (TIL..), so it seems not everybody loves the bot. I've kept it running for now, but I doubt it will survive too much longer.

## Future

Maybe in the future I'll change the filtering to be whitelist based rather than blacklist based and have it only run in subreddits where the moderators approve of it, or only for subreddits that I care about.. so basically only for [r/skateboarding](https://reddit.com/r/skateboarding) and [r/trap](https://reddit.com/r/trap).

Still, I wanted to use this as an opportunity to share some useful information on how to build serverless reddit bots, and how to debug Zappa applications.

Got comments? Shoot me an email to rich [[at]] pubmail [[dot]] io!