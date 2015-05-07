---
layout: post-no-feature
title: "Clarity launch day 'Rendering' bug"
description: "The one where all the keys were bunched up in the top left hand corner..."
category: articles
comments: true
tags: [clojure, webview, clarity, programming]
---

## Context

As this is my first post on the subject, a little context about **Clarity** [(Google Play)](https://play.google.com/store/apps/details?id=com.swiftkey.clarity.keyboard&referrer=utm_source%3Dadamblog%26utm_medium%3Dblog%26utm_content%3Dprogrammingpost): It's a new keyboard brought out by SwiftKey (my employer) as part of our [Greenhouse](http://swiftkey.com/en/greenhouse/) initiative to get internal experiments out in the world and in people's hands. I'm the tech lead (and [Ben Leavett](http://benleavett.github.io/) is the Product Manager). It's pretty exciting tech for a number of reasons - like for example the fact that it is written using [Clojure](clojure.org).

<img src="/images/clarity-icon.png" alt="Clarity logo"/>

The first release was on Monday, and as could be expected the first time it's run on a wide range of different devices and in different situations to those found in the office, there were a few (pretty major) bugs that turned up.

## Ugh, Classloaders

One was an issue with the WebView that we use to render the keyboard - on Samsung and a few other devices, loading the WebView would destroy the classloader for a thread (!?). This resulted in `ClassNotFound` exceptions for anything which tried to load on the same thread in the future - understandable given that it was *looking in the wrong APK*. I believe this is a bug in some implementations of the WebView, and there's nothing we can do about it other than restore the ClassLoader to what it should have been after initialising the WebView.

## The unreproducible rendering bug

The next biggest bug was a bit more perplexing - lots of people were reporting that all the keys simply bunched up into the top left corner of the keyboard like so:

<img src="/images/ClarityLocalisationBug.png" alt="An example of the bug in action"/>

The initial thought was to blame WebView incompatibilities again, so we set about trying to find a pattern in the device and OS versions of everyone this was happening to. Emails were sent around the office asking if anyone had seen this bug, and if so could they bring it for us to debug. If we had a version where this was happening we were pretty sure we could open up the fantastic Chrome Web Development tools with a debug build and diagnose it immediately. However nobody seemed to be getting it in the office, none of my friends in London, nobody could show us this happening!

It took Błażej's cousin Marcin who lives in Poland having the issue to work out why. We sent him a debug build and he pointed out that all the percentage values in the CSS were invalid.

It turns out that in some programmatically generated styles, a too-clever-for-its-own-good string formatting method was helpfully localising our strings for us, so instead of keys being `width:10.00%`, if you happen to live in a country with a different decimal separator you would see `width:10,00%` to which CSS says 'wat?'.

A minor tweak to the formatting method and this one was fixed for everyone. A valuable demonstration that no matter how much testing you do on your own devices, you can never know what's going to go wrong until you've put it out in the world! But that's what releasing this Beta (and the Greenhouse initiative) is all about!

Oh, and if you haven't already downloaded it, go and install Clarity (now with fewer bugs)

