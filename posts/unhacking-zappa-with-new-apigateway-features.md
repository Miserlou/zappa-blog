---
title: Un-Hacking Zappa with New API Gateway Features
author: Rich Jones
date_created: 10-11-2016
format: markdown
---

Okay, I have just published version **0.27.0** of Zappa! If you're a Zappa user, this contains some important changes that you probably want to know about!

The changes stem from the fact that Amazon recently announced some [new features to the API Gateway](https://aws.amazon.com/blogs/aws/api-gateway-update-new-features-simplify-api-development/) that allow us to greatly simplify how Zappa works. We actually got to remove three of the major hacks that powered Zappa previously!

This version contains some pretty major changes to the way that Zappa works, so you will probably want to do a test deployment to make sure that all your app features still work before pushing this new version to production! I think this code needs to be battle-tested a bit more! But, we're a bleeding-edge project, so this is how we do it! Please file issues on the GitHub project if you encounter any issues after upgrading.

### Parameter Mapping

<div class="row" >
    <div class="col-sm-6">
        <img src="https://i.imgur.com/uzyeBfg.png" class="img img-responsive img-thumbnail">
    </div>
    <div class="col-sm-6">
        <img src="https://i.imgur.com/GnWzmyT.png" class="img img-responsive img-thumbnail">
    </div>
</div>

<center>
    <i>Old and busted, new hotness.</i>
</center>

The first major hack was that to ensure that every possible HTTP eventcould be converted into a valid WSGI request, every possible event method was mapped to every possible parameter!

This meant that to deploy an app, we initially had to make *over 1,000 API calls*! We previously replaced those calls with a CloudFormation template, but even that was still over 350Kb of compressed JSON!

Now, we can use the new "ANY" parameter and the "{proxy+}" method to map every single parameter and method with a single entry!

This is much cleaner in our handler code, and it reduced our template size down from 350Kb to 1.5Kb! Awesome!

### Proper Status Codes

Previously, API Gateway didn't allow us to return multiple headers on non-200. This was a problem for certain login forms, (like the Django admin panel) where a POST returns an authentication cookie AND a redirect to the new "logged in page".

As a result, for `text/html` status types, we would return a 200 request with some dynamically inject HTML/JS code in the header that would redirect the user to the correct page. Yuck!

With `{proxy+}`, we no longer have to do this! Status codes are now passed properly. This allows us to close one of our last major [outstanding issues](https://github.com/Miserlou/Zappa/issues/303).

### No More Base64 Madness!

<div class="row" >
    <div class="col-sm-12">
        <img src="https://i.imgur.com/X9othA7.png" class="img img-responsive img-thumbnail">
    </div>
</div>

<center>
    <i>It wasn't pretty, but it worked, dammit!</i>
</center>

Similarly, we had some _major_ hacks (I'm actually pretty proud of them) for mapping status codes to Base64-encoded JSON-ified representations of the HTML responses that we could then re-render and pass through the API Gateway using VTL templates. To make that happen, we had to create our own Python exception class and then raise that through the gateway. Totally freaking crazy.

We can now remove this hack entirely! We're now able to pass anything we like straight through the gateway without raising an artificial exception or any data mangling for any status code we like. Hooray!

### Try it out!

This is a pretty significant change to how Zappa works, but it simplifies a lot of things greatly. There are still a few hacks left (we still have to have our special cookie-encoding middleware because we can't pass multiple cookies simultaneiously through `{proxy+}` yet) - but on the whole everything is a lot cleaner now.

Still, I think it's ready for you to take it for a spin!

If you haven't tried Zappa yet, for any existing Python web application, this is how easy it is to run your own maintainance-free, ultra-scalable, zero-cost server-less deployment:

`$ virtualenv env`

`$ source env/bin/activate`

`(env) $ pip install zappa`

`(env) $ zappa init`

`(env) $ zappa deploy`

`(env) $ echo "Woo! My server is live!"`

Awesome!

### Coming Soon

On the flip side, there is now a lot of cruft in the code, so the next minor version updates will probably just be code removals rather than new features!

Enjoy!

Rich
