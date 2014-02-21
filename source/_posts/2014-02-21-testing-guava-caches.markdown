---
layout: post
title: "On Testing Guava Caches"
date: 2014-02-13 19:27
comments: true
categories: [Programming, Java, Guava]
author: Mark Veronda
---

TLDR: Do not forget your scale when dealing with anything related to time, whether or not you are programming.  Always, always, always say "10 nanoseconds" and never just "10".

I had a frustrating problem at work where I had written a few quick lines of code and found myself finagling the testing for longer than I initially expected. The few lines of code I wrote was a [decorator](link) to add caching functionality.

The caching part was a cakewalk because I knew from the start I would use Google [Guava's]() [amazing]() [library]() that already provides extensive, configurable and battle-tested functionality.  It's just incredible because they literally have thought of everything and I would liek to give a big hat tip to the writers for providing this to the world.  One neat example of their having thought of everything was when I wanted to have a [RemovalListener]() (code to run upon cache eviction) only run on keys that expired due to access time and not keys that are manually evicted.  I spent a few minutes thinking of a solution but when I started to code it up and and took a look at the Guava docs, sure enough, the option is right there and is a simple matter of asking wasEvicted().  Just simply amazing.

The problem I was stuck on was getting the cache eviction to actually happen during testing.  I had followed good [TDD]() practice and wrote my tests ahead of time, but was still failing on the two tests involving a cache eviction.  The second test I wrote was to ensure expiry occurrs after the *first* write for any key, not any subsequent writes. That the RemovalListener was not running and cache eviction not happening at all was painfully obvious by the few missing lines of test coverage.  

It turned out the problem with my test was with my usage of the [Ticker]() interface, which is quite the footnote in the documentation and discussion on testing with Guava's cache. The code I wrote said '+10' to advance time by what I presumed was 10 milliseconds.  I eventually came to the smack-of-the-forehead realization that Ticker is based on nanoseconds and therefore, when advancing its time readout, you need to advance it by TimeUnit.Milliseconds.toNanos(10) nanoseconds (AKA: 10,000,000) and not 10.  Finally, the testboard went green and now I can wir the cache in and wrap up my workweek. 

Normally, wiring in a decorated object takes one minute flat (regardless of whether you're using Spring or Guice), but unfortunately, I'm working in an environment right now that doesn't have any [Dependency Injection (DI)](). Thus, it is as if one was writing code in notepad  with your hands tied behind your back.  Although maybe it's not quite that bad and one could use a modern IDE, but your hands are still tied behind your back. Without DI, it is not easy to casually pass in mocked objects for testing.  At least, that is what it feels like after having been used to DI for quite a while.
