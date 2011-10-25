---
layout: post
title: Installing Reddit From Source.
categories:
  - Reddit
  - Python
---

You may want to check out my post on [installing Reddit for testing](/InstallReddit).

In a [post](http://www.reddit.com/r/redditdev/comments/knkpl/getting_started_with_contributing_good_first/) I made to /r/redditdev about where to start with contributing to Reddit, I got a lot of helpful tips.  One suggestion was that I try fixing [ticket #729](http://code.reddit.com/ticket/729):

"Literal IPv6 addresses in URLs require square brackets around them. reddit reports that these are invalid. For example, try testing it with http://[::1]/, which is a perfectly valid URL."

So I went about getting myself acquainted with the Reddit code base.  Reddit is based on [Pylons](https://www.pylonsproject.org/projects/pylons-framework/about) and I am familiar with anothe web framework [Django](https://www.djangoproject.com/).  My thought process went like this: Reddit is tripping up on its url validation.  So I need to track down the bit of code that verifies urls and correct it.  On the new post submission page, the link you submit is verified.  So if I find that page in the code I can trace my way to where the verification is done.  Pylons has a main routing file, so I can start there.

<script src="https://gist.github.com/1268910.js"> </script>

So in the end the problem was not a part of the Reddit codebase, but in another library.    

~Matt
