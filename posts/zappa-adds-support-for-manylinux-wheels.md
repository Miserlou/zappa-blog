---
title: Zappa Adds Support for Manylinux Wheels
author: Rich Jones
date_created: 11-17-2016
format: markdown
---

New feature alert!

Zappa now has out-of-the-box support for AWS Lambda compatible "manylinux" wheels! This means that *[hundreds more](https://gist.github.com/perrygeo/9545f94eaddec18a65fd7b56880adbae)* Python libraries with C-extensions will now work with Zappa with no additional work!

Some choice goodies include, but aren't limited to:

* bcrypt

* cffi

* pandas

* gevent/greenlet

* lxml

* matplotlib

* pymongo

* pyzmq

* scikit-learn

* scipy

* secp256k1

This expands upon our [lambda-packages](https://github.com/Miserlou/lambda-packages) effort, which houses pre-compiled and pre-optimized Lambda-compatible version of many popular python packages and supplements them with versions already compiled for 64-bit linux and housed on PyPi.

As usual, this is a brand-spanking-new feature, so please use it at your own risk and report any issues you may find! You can disable it by setting `use_precompiled_packages` to `false` in your `zappa_settings` file.

Special thanks to *[@perrygeo](https://github.com/Miserlou/Zappa/issues/398)* for suggesting this feature and providing a prototype implementation.

Enjoy!,

Rich