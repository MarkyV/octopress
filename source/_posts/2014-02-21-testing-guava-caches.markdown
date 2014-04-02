---
layout: post
title: "On Testing Guava Caches and Time"
date: 2014-04-02 00:00
comments: true
categories: [Programming, Java, Guava, NotesToSelf]
author: Mark Veronda
---

TLDR: Do not forget your scale when dealing with anything related to time, whether or not you are programming.  Always, always, always say "10 nanoseconds" and never just "10".

I had a frustrating problem where I had written a few quick lines of code and found myself finagling the testing for longer than I initially expected. The lesson I learned, as provided in the TLDR, actually came from a book I was reading at the same time called ["In Search of Certainty"](http://markburgess.org/certainty.html).  The book is written by Mark Burgess, and I must say that I am shocked at how few people know about one of the most foreword thinking Computer Scientists currently around.  Equally as few people have heard of [CFEngine](https://www.youtube.com/watch?v=4CCXs4Om5pY), which should not need introduction as it was literally 10 years ahead of everyone else.  The few lines of code I wrote was a decorator to add caching functionality.

The caching part was a cakewalk because I knew from the start I would use Google [Guava's](https://code.google.com/p/guava-libraries/) [amazing](http://stackoverflow.com/a/4543114) [library](http://gubendran.blogspot.com/2012/06/google-guava-collections-are-awesome.html) that already provides extensive, configurable and battle-tested functionality.  I would like to give a big tip of the hat I don't wear to the authors of the library for providing this to the world.  One neat example of how well thought out the library is when I wanted to have a [RemovalListener](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/RemovalListener.html) (code to run upon cache eviction) only run on keys that expired due to access time and not keys that are manually evicted.  I spent a few minutes thinking of a solution but when I started to code it up and took another look at the Guava docs, sure enough, the option is right there and is a simple matter of asking the RemovalNotification object wasEvicted().  Just simply amazing.

The problem I was stuck on was getting the cache eviction to actually happen during testing.  Since I saw a good opportunity to apply TDD here, I actually did write my tests ahead of time.  Which was a fortunate thing because I knew I wasn't finished due to the two failing tests involving a cache eviction.  The second test I wrote was to ensure expiry occurs after the *first* write for any key, not any subsequent writes. That the RemovalListener was not running and cache eviction not happening at all was painfully obvious by the few missing lines of test coverage.  

It turned out that the problem with my tests was with my usage of the [Ticker](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Ticker.html) interface, which is quite the footnote in the documentation and discussion on testing with Guava's cache. The code I wrote said '+10' to advance time by what I presumed was 10 milliseconds.  I eventually came to the smack-of-the-forehead realization that Ticker is based on nanoseconds and therefore, when advancing its time readout, you need to advance it by TimeUnit.Milliseconds.toNanos(10) nanoseconds (AKA: 10,000,000) and not 10.  Finally, the testboard went green quickly followed by wiring up the cache and thus wrapping up my workweek in a New York minute. 
