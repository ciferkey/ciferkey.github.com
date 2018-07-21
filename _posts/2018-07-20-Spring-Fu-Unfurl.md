---
layout: single
title: Spring Fu Mini Project: Link Unfurling API
categories:
  - Projects
---

> Every programmer occasionally, when nobody’s home, turns off the lights, pours a glass of scotch, puts on some light German electronica, and opens up a file on their computer. It’s a different file for every programmer. Sometimes they wrote it, sometimes they found it and knew they had to save it. They read over the lines, and weep at their beauty, then the tears turn bitter as they remember the rest of the files and the inevitable collapse of all that is good and true in the world.

From [Programming Sucks](https://www.stilldrinking.org/programming-sucks)

I have been messing around with a new Spring project called [Spring Fu](https://github.com/spring-projects/spring-fu). There is a nice [introduction video](https://spring.io/blog/2018/06/13/spring-tips-spring-fu) but here are some of the basic points that stood out to me about Spring Fu:
 * Its an experimental test bed for new Spring Boot features. Currently pre 0.0.1 so many things are still in flux.
 * Takes Spring 5 as a base (See [Whats new in Spring 5](https://spring.io/blog/2016/09/22/new-in-spring-5-functional-web-framework) if you haven't looked yet at whats new)
 * Makes a micro-framework (designed to be light weight rather than cover every possible need)
 * Using the language Kotlin (made by JetBrains, [has some nice improvements over Java](https://kotlinlang.org/docs/reference/comparison-to-java.html))
 * Designed around [Functional Configuration](http://www.baeldung.com/spring-5-functional-beans) (yet another way to configure Spring applications, but this time trying to minimize reflection).
 * Built off of [Webflux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) for reactive programming (and there is work in progress to adapt this to Kotlin's [coroutines](https://kotlinlang.org/docs/reference/coroutines.html)

The project builds off the minimal reactive example from the github project. I had briefly touched Kotlin and Gradle prior to this for with Android but this was my first time working with the [Kotlin Gradle DSL](https://blog.gradle.org/kotlin-meets-gradle). As always, [the code](https://gitlab.com/ciferkey/fu-unfurl) is available for viewing.

I wanted a sample project to play with so I modeled the project as a simple API for [link unfurling](https://medium.com/slack-developer-blog/everything-you-ever-wanted-to-know-about-unfurling-but-were-afraid-to-ask-or-how-to-make-your-e64b4bb9254) and tried to test out the common things one might use in an application along the way (logging, metrics, etc).

The high level overview of the unfurler is:
	1. fetch the requested page
	2. if the url matches a custom unfurler use the matched unfurler (eg if the site is youtube use the youtube embed unfurler)
	3. if metadata in the head section contains tags for [opengraph](http://ogp.me/) or [Twitter Cards](https://developer.twitter.com/en/docs/tweets/optimize-with-cards/overview/markup.html) use that information
	4. otherwise use default unfurler which uses the first paragraph as the descript and the first image as the thumbnail
	5. Return JSON response containing metadata, title, thumbnail, and body

As I had mentioned Spring Fu is still very new and there were some bugs I encountered but the [Kotlin Slack](http://slack.kotlinlang.org/) was super helpful for working through them:
* I couldn't import webflux-jackson to handle configuring json decoding/encoding. [Got a response](https://kotlinlang.slack.com/archives/C0B8ZTWE4/p1531774520000016?thread_ts=1531767215.000097&cid=C0B8ZTWE4) to my question in the #spring channel and then a swift [fix](https://github.com/spring-projects/spring-fu/commit/798b9dfd347d18e4c7683c86575183fbb708d0b1) in the project repository.
* There was a bug around having a separate class for handling routing and it seems its [due to a recent refactor](https://kotlinlang.slack.com/archives/C0B8ZTWE4/p1532095680000287), but I [got a response](https://kotlinlang.slack.com/archives/C0B8ZTWE4/p1532104961000327) and a [fix](https://github.com/spring-projects/spring-fu/commit/f7309ac0c11659fc3acd16081ba841a78ae398a6) was made.

Overall I enjoyed the experience and am excited to try out another project once 0.0.1 lands!
