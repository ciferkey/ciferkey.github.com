---
layout: single
classes: wide
title: "Google Cloud Platform with Gradle Kotlin DSL"
description: "Converting the Groovy Gradle configuration for Google Cloud Platform to the Kotlin DSL"
categories:
  - Projects
---

I was trying to deploy a Spring application to App Engine [as per the documentation](https://cloud.google.com/community/tutorials/kotlin-springboot-app-engine-java8) and realized that while the GCP documentation [covers the Groovy based DSL for Gradle](https://cloud.google.com/appengine/docs/standard/java/using-gradle) it doesn't have any official documentation for the newer Kotlin DSL.

Bits and pieces were available online but it took some time to piece it all together. I'm sharing it her in hopes it can save others from having to figure it out again. For those that just want to grab the Kotlin DSL version is here:

{% highlight kotlin %}
{% raw %}

import org.jetbrains.kotlin.gradle.tasks.KotlinCompile
import com.google.cloud.tools.gradle.appengine.standard.AppEngineStandardExtension

buildscript {
	repositories {
		jcenter()
		mavenCentral()
	}
	dependencies {
		classpath("com.google.cloud.tools:appengine-gradle-plugin:2.4.1")
	}
}

plugins {
	war
	kotlin("jvm") version "1.4.21"
	kotlin("plugin.spring") version "1.4.21"
	kotlin("plugin.jpa") version "1.4.21"
}

apply(plugin = "com.google.cloud.tools.appengine")

group = "com.example"
version = "0.0.1-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_11

repositories {
  maven {
		url = uri("https://oss.sonatype.org/content/repositories/snapshots")
	}
	mavenCentral()
	jcenter()
}

extra["springCloudGcpVersion"] = "2.0.0"
extra["springCloudVersion"] = "2020.0.0"

dependencies {
	implementation("com.google.appengine:appengine-api-1.0-sdk:+")  // Latest App Engine Api's
	providedCompile("javax.servlet:javax.servlet-api:4.0.1")
	implementation("jstl:jstl:1.2")

  // Add your dependencies here.

	// Testing
	testImplementation("junit:junit:4.12")
	testImplementation("com.google.truth:truth:1.1")
	testImplementation("org.mockito:mockito-all:1.10.19")
	testImplementation("com.google.appengine:appengine-testing:+")
	testImplementation("com.google.appengine:appengine-api-stubs:+")
	testImplementation("com.google.appengine:appengine-tools-sdk:+")
}

dependencyManagement {
	imports {
		mavenBom("com.google.cloud:spring-cloud-gcp-dependencies:${property("springCloudGcpVersion")}")
	}
}

tasks.withType<com.google.cloud.tools.gradle.appengine.core.DeployTask> {
	dependsOn("test")
}

tasks.withType<com.google.cloud.tools.gradle.appengine.standard.StageStandardTask> {
	dependsOn("test")
}

tasks.withType<KotlinCompile> {
	kotlinOptions {
		freeCompilerArgs = listOf("-Xjsr305=strict")
		jvmTarget = "11"
	}
}

// https://stackoverflow.com/a/60225081/564606
tasks {
	test {
		useJUnitPlatform()
		testLogging.showStandardStreams = true
		beforeTest(closureOf<TestDescriptor> {
			logger.lifecycle("test: $this  Running")
		})
		onOutput(KotlinClosure2<TestDescriptor,TestOutputEvent,Unit>({descriptor, event ->
			logger.lifecycle("test: " + descriptor + ": " + event.message )
			Unit
		}))
		afterTest(KotlinClosure2<TestDescriptor,TestResult,Unit>({descriptor, result ->
			logger.lifecycle("test: $descriptor: $result")
			Unit
		}))
	}
}

// https://github.com/GoogleCloudPlatform/app-gradle-plugin/issues/191#issuecomment-462404681
the<AppEngineStandardExtension>().apply {
	deploy {   // deploy configuration
		projectId = System.getenv('GOOGLE_CLOUD_PROJECT')
		version = "1"
	}
}

{% endraw %}
{% endhighlight %}

# Figuring this all out

## Buildscript

{% highlight kotlin %}
{% raw %}

buildscript {    // Configuration for building
  repositories {
    jcenter()    // Bintray's repository - a fast Maven Central mirror & more
    mavenCentral()
  }
  dependencies {
    classpath 'com.google.cloud.tools:appengine-gradle-plugin:2.2.0' // If a newer version is available, use it
  }
}

{% endraw %}
{% endhighlight %}

There are some general Groovy vs Kotlin syntax changes we will need to handle here and in the rest of the code:
 - single quotes need to become double quotes
 - methods like classpath, implementation, testImplementation need parenthesis around them.

{% highlight kotlin %}
{% raw %}

buildscript {
	repositories {
		jcenter()
		mavenCentral()
	}
	dependencies {
		classpath("com.google.cloud.tools:appengine-gradle-plugin:2.4.1")
	}
}

{% endraw %}
{% endhighlight %}


## Plugins
{% highlight groovy %}
{% raw %}

apply plugin: 'java'                              // standard Java tasks
apply plugin: 'war'                               // standard Web Archive plugin
apply plugin: 'com.google.cloud.tools.appengine'  // App Engine tasks

{% endraw %}
{% endhighlight %}


## Adding the plugin
Originally I had wanted to use the new "plugins" syntax for adding the appengine plugin but it seems that [it only works if the plugin is available in the Gradle Plugins Repository](https://stackoverflow.com/a/32353244/564606). So we have to use the older "apply" syntax instead.


{% highlight kotlin %}
{% raw %}

plugins {
	war
	kotlin("jvm") version "1.4.21"
	kotlin("plugin.spring") version "1.4.21"
	kotlin("plugin.jpa") version "1.4.21"
}

apply(plugin = "com.google.cloud.tools.appengine")

{% endraw %}
{% endhighlight %}


##
{% highlight groovy %}
{% raw %}



{% endraw %}
{% endhighlight %}

{% highlight kotlin %}
{% raw %}



{% endraw %}
{% endhighlight %}

## Other changes
I updated the plugin version as the comment in the original example suggests. You should too if you use this.

## Adding the dependencies
In the Groovy DSL you can use deprecated "compile" syntax for adding dependencies as well as the newer "implementation" syntax. We can only use the later in the Kotlin DSL.

## Deploy and stageStandard tasks
[This answer](https://stackoverflow.com/a/60225081/564606) on Stack Overflow explains the syntax I used for getting the plugin's deploy task and stageStandard task to depend on the test task.

## Deploy block
[This issue](https://github.com/GoogleCloudPlatform/app-gradle-plugin/issues/191#issuecomment-462404681) on Github explains the necessary syntax for the final deploy block.


https://github.com/passsy/kotlin-dsl/blob/master/doc/getting-started/Closures.md

needs to be of type TestDescriptor https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.Test.html

https://nwillc.wordpress.com/2019/01/08/gradle-kotlin-dsl-closures/


https://stackoverflow.com/a/55849201/564606 and gradle log "
The task 'appengineDeploy' (com.google.cloud.tools.gradle.appengine.core.DeployTask) is not a subclass of the given type (org.gradle.api.tasks.compile.JavaCompile).
"

