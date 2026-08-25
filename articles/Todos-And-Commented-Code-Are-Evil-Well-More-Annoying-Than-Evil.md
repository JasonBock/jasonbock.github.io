---
title: TODOs and Commented Code Are Evil ... Well, More Annoying Than Evil
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# TODOs and Commented Code Are Evil ... Well, More Annoying Than Evil

I've noticed at the current client that they tend to use TODOs in the code base more frequently in the code base than I've seen in the past. The coders also tend to comment blocks of code out, rather than just deleting it.

This is getting a bit annoying. It's not that I don't do these things now and then; I'm realizing how thorny these aspects are.

First, TODOs. They're usually added because the developer stumbled upon an implementation detail or a fuzzy customer request and they can't go on from there. The problem is, TODOs are easily forgotten (and yes, I know VS .NET can show TODOs in a list). I'm not against TODOs, but the issue is when they end up littering the code and no one ever addresses them. By the time you get around to addressing them, features may have changed or the TODO commentary isn't sufficient enough to understand what needs to be done.

Second, commented code. This annoys me, especially when you're using a source control system. When I see code with lots of commented code, I start to wonder, why is this code commented out? Why didn't the developer just delete it? I've seen this over and over on many old code bases, and it's disheartening because you don't know why the code is commented out. This is really strange when you use source control because you can always go back in the source tree and get the code back that you want.

OK, rant off. They're minor things, but they bug me.

> Published: 11.11.2005 03:12:50 PM CST