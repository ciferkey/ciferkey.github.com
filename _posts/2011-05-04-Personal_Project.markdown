---
layout: post
title: Personal Project
categories:
  - Django
  - Python
  - MongoDB
  - TL;DR
---
**Ingress**

I was in Cancun, on vacation, in the middle of sixth grade.  Rather than sit idle and soak up the sun I bought my first programming book and dove in.  Since then I have been a voracious reader when it comes to may area of okatu: computer science.  The following is a brief overview of a personal project I have been working on, that I intend to demonstrate some of the abilities I have accrued so far.

**Case Study**

*Project Goal #1* To develop a site that aggregates cross country race results, data mines them and presents them with insights.

*Project Goal #2* To train myself in the use of several web technologies and development tools.

Initially I began developing several separate components in python.  The first of which was an aggregator which would crawl the Rhode Island Track and Field website, pull the race results as text files, and store their contents in the database. The code initially interfaced with MongoDB directly through queries using pymongo.  I eventually migrated to using MongoKit as an ORM, but I still had to manage much of the work manually.

The second component was a sanitization mechanism for the race results.  Races are not reported in a standard format and I needed them in a formalized manner to work with.  This was actually the most difficult logical component, figuring out generalized patterns that the information could follow.  I employed heavy use of regular expressions to get this functioning.  I have results from 2005 to 2009, an example file can be seen [here](http://ritca.com/results/2009-2010/Cross-Country/Boys/2009-11-08.State%20Championship.txt), and I am working on adding the 2010 season, which unfortunately are stored in pdfs.

While working on this I quickly found it difficult to synchronize my work between my multiple work stations (a virtualized instance of Ubuntu and a netbook modified to run OS X).  I began using git and Github for this purpose but quickly found the actual intention of version control to be much more useful.  The code security and ability to track changes, branch, and merge was revolutionary.

The third component consisted of a simple parser that would go through the sanitized race results (stored in JSON) and store the information in the database.  It was interesting constructing a schema for nonrelation database. In retrospect I was stuck in the mindset of a relation database.  This caused me to work with foreignkeys rather than embedding objects and after research it would appear that my current schema would not scale well (though I have read and agree with [37signal’s](http://gettingreal.37signals.com/ch04_Its_a_Problem_When_Its_a_Problem.php) approach: Its more important to have working code, than an unimplemented idea. After all you can't feel too bad about being overrun with users).

At this point the data was all loaded into the database and I began working on working on a prediction engine.  I have been experimenting with several different approaches to models runners’ growth, but currently have not found a satisfactory one. Most of the statistical analysis is done using either numpy or scipy for performance reasons. This is currently the most active area of development.

The final component I worked on was creating a web application with Django to display the information. Initially I had wanted to use Pylons, but it’s no longer in development and its successor Pyramid is still too young to use.  It was also very interesting to use Django with a nonrelation database.  However it has a backend that can plug in nearly seamlessly (once all the quirks are figured out). Interestingly enough after using Django's wonderful ORM I decided to integrate all my existing code into a single project under Django. The fact that Django can use a nonrelational database so transparently meant that it hardly felt like I was a nonrelational database at all (which feels like it almost defeats the purpose of it, except for the one upside of not having to deal with migration). I also implemented dynamic graph generation for the site with matplotlib.  I could generate svg images, cache them in the database and directly embed them thanks to html5.

I was able to get free hosting on [Ep.io](http://www.ep.io/) (one of the current contenders for the Heroku of Python),  [MongoLab](https://mongolab.com/) and [PostageApp](http://postageapp.com/). As well as a free student account on Github (its amazing what one can get for just asking).

The (rough) beta for the site is available at runnr.ep.io however many features still need to implemented. Here are a [couple](http://runnr.ep.io/) [example](http://runnr.ep.io/runner/2011/Matt/Brunelle/) [pages](http://runnr.ep.io/race/2008/11/02/State%20Championship).  It is intended to be more of a mechanism for experimentation, than an actual product.

**What's Next**

My next goal is to improve the quality and consistence of my code.  I am going to focus on study coding styles (specifically PEP 8), programming with more than one developer, bug tracking, ticket systems, basic web design and agile development.

**Future Education**

I will be attending the University of Massachusetts Amherst for Computer Science this coming Fall.

~Matt

