---
layout: single
classes: wide
title: "Glassistant Google Assistant on Google Glass Part I"
description: "Bringing Google Assistant to the Google Glass using the Java Assistant SDK"
categories:
  - Projects
---

_Note: This port isn't really complete but its been sitting on the backburner for over a year now so I'm pushing it out as a draft._

Google announced [Enterprise Edition 2](https://www.blog.google/products/hardware/glass-enterprise-edition-2/) of Google Glass. I'm very jealous I cannot buy one but that is probably for the best considering I bought a used Google Glass XE-C (the last Glass revision available to consumers) off of Ebay back in February of 2018 and never finished my project with it...

The goal of the project was to add Google assistant connectivity to the Google Glass. At the time (end of 2017) Google had just [announced](https://developers.googleblog.com/2017/12/the-google-assistant-sdk-new-languages.html) updates to the Google Assistant SDK that allowed additional ways to call the assistant API. My plan was to adapt the examples given for running assistant on Android Things to instead run on the Glass. I decided to call the project Glassistant because it simultaneously breaks the branding guidelines both [Glass](https://developers.google.com/glass/distribute/branding-guidelines) and [Assistant](https://developers.google.com/actions/policies/branding-policies). 

### Previous Implementation
Back when I had implemented the project in 2018 I was able to issue Assistant requests over the API:
![First successful assistant SDK request from glass](/images/first-request.png)
However the project was lacking in a couple areas and Glass runs an older version of Android (4.4 KitKat) which complicates matters. The primary areas I wanted to address this time are:
- There was a [bug in GRPC](https://github.com/grpc/grpc-java/pull/3971) that prevented it from using the correct Security Provider. I had to depend on a custom build of GRPC while waiting for the changes to be released.
- There was no pure voice driven invocation like on normal android devices. You had to first say "Ok Glass" to bring up the voice triggers menu and then say "ask Assistant....".
- There was essentially no user interface. It would display the response from Assistant on a Glass Card but there was no clear flow and no way to make multiple requests without restarting. 

### On the problem with Security Providers in GRPC:
Android 4.4 KitKat does not come with newer Java security providers required for GRPC. Normally the recommended way to [update security providers](https://developer.android.com/training/articles/security-gms-provider) on Android is through Google Play Services. Unfortunately play services is not available for Glass. Instead GRPC [recommends using](https://github.com/grpc/grpc-java/blob/master/SECURITY.md#bundling-conscrypt) the conscript library by Google and manually registering the new security provider. Recent versions of GRPC-Java will now use conscrypt and you no longer need a custom build.

### On the problem of voice invocation and cleaner UX
This is still a work in progress and enough work that I'd like to provide more detail in a future post.

### Porting Process
The Glass SDK is still available for use in Android Studio. In the Android SDK section of the settings insure you have installed the 4.4 (KitKat) SDK Platform. It doesn't mention Glass but if you check the "Show Package Details" checkbox you will see it includes "Glass Development Kit Preview".
![Android Studio SDK settings for glass](/images/sdk-settings.png)

Next I created a new project starting with the Glass Immersion Activity as the base.
![Android Studio SDK settings for glass](/images/glass-immersion-activity.png)

To issue unregistered Glass voice commands during development you need to [add a permission](https://developers.google.com/glass/develop/gdk/voice#unlisted_commands) to the manifest.

The assistant integration is derived from [sample Google Assistant project for Android Things](https://github.com/androidthings/sample-googleassistant). I first started by [migrating the GRPC module](https://github.com/ciferkey/glassistant/commit/23ad68bb16ef7f0a2f3ae849e41e1e33fdc58f63) over into the base Glass project. I also had to [move to a newer version](https://github.com/ciferkey/glassistant/commit/0b71f58d572de31077f05dc15121fdf3a45f3d60#diff-5b3066d7378782045d33568340c02ebb) of GRPC to fix the security provider bug.

Next step was adapting the SDK integration they had to work on Glass. The [EmbeddedAssistant](https://github.com/androidthings/sample-googleassistant/blob/master/app/src/main/java/com/example/androidthings/assistant/EmbeddedAssistant.java) class provides most of the heavy lifting but there are some Android Things related pieces that need to be excised since the demo can invoke assistant with a button press. Additionally since the Glass is running 4.4 there are are number of audio api updates (AudioDeviceInfo, AudioTrack, etc) that are not available and need to be replaced with calls for the older API.

### Testing
There is no way to emulate Glass you can only test by running directly on the device. You also need to [enable debug](https://stackoverflow.com/questions/21542577/how-can-we-enable-debugging-mode-on-google-glass-for-testing-an-android-app-on-g) on your Glass. Also since I was developing on Windows 10 I needed to [perform a fix](https://stackoverflow.com/questions/20435778/google-glass-is-not-listed-as-android-device-by-adb/42312419#42312419) for adb to pick up Glass.
