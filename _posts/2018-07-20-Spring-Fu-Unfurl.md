---
layout: single
classes: wide
title: "Spring Fu Test Project: Link Unfurling Service"
categories:
  - Projects
---

<style type="text/css">
  .gist-meta {
    display: none;
  }
</style>

> Every programmer occasionally, when nobody’s home, turns off the lights, pours a glass of scotch, puts on some light German electronica, and opens up a file on their computer. It’s a different file for every programmer. Sometimes they wrote it, sometimes they found it and knew they had to save it. They read over the lines, and weep at their beauty, then the tears turn bitter as they remember the rest of the files and the inevitable collapse of all that is good and true in the world.
> <cite>From [Programming Sucks](https://www.stilldrinking.org/programming-sucks)</cite>


### Spring Fu

I have been messing around with a new Spring project called [Spring Fu](https://github.com/spring-projects/spring-fu). Pivotal has a nice [introduction video](https://spring.io/blog/2018/06/13/spring-tips-spring-fu) for the project. The key takeaways about Spring Fu are:
 * Its an experimental test bed for new Spring Boot features. The project is currently pre 0.0.1 so many things are still in flux.
 * Takes Spring 5 as a base (See [Whats new in Spring 5](https://spring.io/blog/2016/09/22/new-in-spring-5-functional-web-framework) if you haven't looked yet at what has changed)
 * The project is designed to be a lightweight micro-framework with low resource usage and fast startup.
 * Its written in Kotlin, the language made by JetBrains which has some [nice improvements over Java](https://kotlinlang.org/docs/reference/comparison-to-java.html).
 * Provides a DSL leveraging [Functional Configuration](http://www.baeldung.com/spring-5-functional-beans) which helps reduce the amount of reflection used.
 * Built off of [Webflux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) for reactive programming and there is work in progress to adapt this to work with Kotlin's [coroutines](https://kotlinlang.org/docs/reference/coroutines.html).

The project builds off the minimal reactive example from the github project. I had briefly touched Kotlin and Gradle prior to this project when on an Android project but this was my first time working with the [Kotlin Gradle DSL](https://blog.gradle.org/kotlin-meets-gradle). 

As always, [the code](https://gitlab.com/ciferkey/fu-unfurl) is available for viewing.

### Link Unfurling Example Project

I wanted a small and focused idea to test Spring Fu with. I decided to create a simple API for [unfurling links](https://medium.com/slack-developer-blog/everything-you-ever-wanted-to-know-about-unfurling-but-were-afraid-to-ask-or-how-to-make-your-e64b4bb9254).

The high level overview of the unfurling logic is:
 1. Fetch the requested page. This is the only step that involves I/O and it should be done reactively so its non blocking.
 2. If metadata in the head section contains tags for [OpenGraph](http://ogp.me/) or [Twitter Cards](https://developer.twitter.com/en/docs/tweets/optimize-with-cards/overview/markup.html) use that information
 3. Otherwise use default unfurler which uses the first paragraph as the description and the first image as the thumbnail
 4. Return JSON response containing title, thumbnail, and description

For example if you requested to unfurl "http://www.baeldung.com/spring-reactor" you would get back the following response:
<script src="https://gist.github.com/ciferkey/3a592c4925436aa97b23fee3609f2953.js"></script>
In this case there was OpenGraph meta data that was used to build the response.

The application configuration ended up very similar to the sample configuration for the project except that I configure beans specific to my handler and I also add Jackson as a codec since I'm producing a JSON response:
<script src="https://gist.github.com/ciferkey/41bda185b23f68566592f4f727b23c4a.js"></script>

This sample project is small so there is only one API endpoint I have setup "/unfurl/{url}". The sample code had inlined the routing logic into the main configuration but I chose to separate it like you would in a larger project. Additionally the routes do not sit inside a class since Kotlin allows top level methods. 
<script src="https://gist.github.com/ciferkey/abb103f5c3529f6987288b399dc4343b.js"></script>

Handlers in Webflux take a ServerRequest which contains all the request information and the handlers generate a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)&lt;ServerResponse&gt;. Internally the handler is generating a Mono of the fetched document using [Spring Webclient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html) which is the reactive equivalent of the traditional RestTemplate. (As a side note, if I had needed to call another API rather than just fetch a page I would be tempted to use [Feign](https://github.com/OpenFeign/feign). It seems like there is current [work in progress](https://github.com/spring-cloud/spring-cloud-openfeign/pull/11) to create a reactive version of Feign which I definitely would like to try out at some point in the future). Finally I've omitted the logic for [extracting the document information](https://github.com/ciferkey/spring-fu-unfurl/blob/master/src/main/kotlin/com/matthewbrunelle/blog/Extractors.kt) since it is not Spring Fu specific.
<script src="https://gist.github.com/ciferkey/212e9a5720744dff5754e34f43791577.js"></script>

### Interacting with the developers behind the project

As I had mentioned Spring Fu is still very new and there were some bugs I encountered but the [Kotlin Slack](http://slack.kotlinlang.org/) was super helpful for resolving through them:
* I couldn't import webflux-jackson to handle configuring json decoding/encoding. [I got a response](https://kotlinlang.slack.com/archives/C0B8ZTWE4/p1531774520000016?thread_ts=1531767215.000097&cid=C0B8ZTWE4) to my question in the #spring channel and then a swift [fix](https://github.com/spring-projects/spring-fu/commit/798b9dfd347d18e4c7683c86575183fbb708d0b1) in the project repository.
* There was a bug around having a separate class for handling routing and it seems its [due to a recent refactor](https://kotlinlang.slack.com/archives/C0B8ZTWE4/p1532095680000287), but I [got a response](https://kotlinlang.slack.com/archives/C0B8ZTWE4/p1532104961000327) and a [fix](https://github.com/spring-projects/spring-fu/commit/f7309ac0c11659fc3acd16081ba841a78ae398a6) was made.

Overall I enjoyed the experience and am excited to try out another project once 0.0.1 lands!
