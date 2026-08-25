---
title: Sql Profiler Nailed It
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Sql Profiler Nailed It

I was running some tests on my application, specifically to see if we needed any tuning on the database side. After running a trace I ran it through the Index Tuning Wizard and, once it was done crunching the numbers, it suggested a couple of index changes. The key point is that it said I should expect a 37% increase in performance once the changes were made.

Guess how much my performance increased by when I made the changes?

37%.

**WOW.**

For some reason, that shocks the hell out of me. I've always been suspect of wizards and what they guesstimate, but this time it got it completely right.

Of course, one could argue that a broken clock is right twice a day, but ... whatever. I'm impressed.

> Published: 09.12.2005 02:50:56 PM CST