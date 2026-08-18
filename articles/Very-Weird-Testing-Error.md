---
title: Very Weird Testing Error
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Very Weird Testing Error

I just ran into a scenario where my unit test would run if I ran it with the debugger attached, but it would fail without the debugger. What was really weird was when I added a `Debugger.Break()` call in the code - take a look at this screen shot:

![Weird Testing Error](https://jasonbock.net/images/Weird-Testing-Error.png "Weird Testing Error")

It seems like the call gets "off" by a couple lines of code, so I end up getting a `NullReferenceException`.

What I did to fix this was to turn Code Coverage off. Seems like that was screwing it up. But I've never seen this happen before.

This is VS 2008 Team Edition - Software Developer.

Any ideas as to why this is happening?

UPDATE: The code is definitely different when Code Coverage is on, but it appears to be the same in terms of functionality. But something is just not getting with that instrumentation in place ...

UPDATE: If I only turn Code Coverage on for two assemblies, I have no problems. Only when the 3rd one is covered does the error occur. Very, very ... weird ...

> Published: 04.09.2008 01:45:16 PM CST