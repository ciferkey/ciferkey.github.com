---
layout: post
title: Installing Reddit From Source On OS X.
categories:
  - Reddit
  - Python
---

I have been looking to an open source project to contribute to and [Reddit](http://www.reddit.com) seems to be the perfect match.  Its code base is in Python (and Pyrex), which my the language I primarily use, its based on [Pylons](https://www.pylonsproject.org/projects/pylons-framework/about) which is very similar to the web framework I use [Django](https://www.djangoproject.com/), and they use [Git](http://git-scm.com/) for version control as well.

If I wanted to do the developement on Ubuntu, there is an automated install script.  Otherwise for OS X, I had to manually install all the dependancies and set them up myself. Based off of the [install guide](https://github.com/reddit/reddit/wiki/Install-guide) and [dependancy guide](https://github.com/reddit/reddit/wiki/Dependencies) I only had to do some minor tweeking to get things up and running.

[Homebrew](http://mxcl.github.com/homebrew/) made installing most of the dependancies easy, however Reddit was picky about what version was install for some of them.  It also can only install the newest version of a piece of software.  Since versions numbers are always going to be changing, I am not going to say which versions you need specifically.  One library that deffinetely needs to be the correct version stated on the dependancy guide is Thrift.  Luckily it installs from source once you install its dependancies.

#Pip and getting the right c extensions
#setting up postgress user and hiding it from login and context menu.


~Matt
