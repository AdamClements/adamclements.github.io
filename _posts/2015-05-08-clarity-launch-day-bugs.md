---
layout: post-no-feature
title: "Clarity launch day bugs"
description: "Why Clarity wouldn't load on Samsung phones and wouldn't render in Poland!"
category: articles
comments: true
tags: [clojure, webview, clarity, programming]
---

## Context

This post refers to [Clarity Keyboard](https://play.google.com/store/apps/details?id=com.swiftkey.clarity.keyboard&referrer=utm_source%3Dadamblog%26utm_medium%3Dblog%26utm_content%3Dprogrammingpost). For a more general overview of that project and its tech see our [previous blog post](/articles/clarity-keyboard-uses-clojure/).

The first public release of our new experimental keyboard Clarity was last week. As could be expected the first time it's run on a wide range of different devices and in different situations to those found in the office there were a few (pretty major) bugs that turned up.

## Ugh, ClassLoaders

The biggest bug was an issue with the view renderer we use to display the keyboard. On Samsung devices as well as a few others, loading the stock renderer destroys the ClassLoader for that thread (!??) replacing it with a custom Samsung ClassLoader. This results in `ClassNotFound` exceptions being raised whenever anything tries to load something on the same thread in the future - which in unsurprising given that the WebView has overridden the ClassLoader only to look in its own APK and no longer looks in ours at all! I believe this is a bug in some implementations of (Google's) stock renderer on Android and there's nothing we can do about it other than save the original ClassLoader before loading the renderer and then manually restore it immediately after its initialisation. Not a pretty fix but the only one we found to work.

## The unreproducible rendering bug

The next biggest bug was a bit more perplexing - lots of people were reporting that all the keys simply bunched up into the top left corner of the keyboard like so:

<img src="/images/ClarityLocalisationBug.png" alt="An example of the bug in action"/>

The initial thought was to blame our view renderer incompatibilities again, so we set about trying to find a pattern in the device and OS versions of everyone this was happening to. Emails were sent around the office asking if anyone had seen this bug, and if so could they bring it for us to debug. Without a way to reproduce it was going to be a tough bug to diagnose, never mind fix. However nobody seemed to be getting the problem in the office, nor amongst my friends in London (where our office is based)!

It took Błażej's cousin Marcin who reported the issue and lives in Poland to help us work out why only some of our users were seeing this bug. We sent him a debug build and he pointed out that all the percentage values in the keyboard layout dimensions were invalid.

It turns out that in some programmatically generated styles, a too-clever-for-its-own-good string formatting method was helpfully localising our strings for us. So instead of keys being `width:10.00%` if you happen to live in a country that doesn't use '.' as a decimal separator you would see something like `width:10,00%` - to which the renderer says 'wat??'.

A minor tweak to the formatting method and this one was fixed for everyone. A valuable demonstration that no matter how much testing you do on your own devices, you can never know what's going to go wrong until you've put it out in the world! But that's what releasing this Beta (and the Greenhouse initiative) is all about!

Oh, and if you haven't already downloaded it go and [install Clarity](https://play.google.com/store/apps/details?id=com.swiftkey.clarity.keyboard&referrer=utm_source%3Dadamblog%26utm_medium%3Dblog%26utm_content%3Dprogrammingpost) (now with fewer bugs).

