---
title: Aborting Threads
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Aborting Threads

I've seen a couple of postings on aborting threads in .NET. Since the thread abortion can take place at any time, it can happen in a type initializer. Therefore, an exception occurs in the initializer, and when this happens, the type can no longer be loaded in that AppDomain. This came up when I and another guy were working on a cancellable thread aspect to our system. Everything was working fine (even with our unit tests - just shows that even if your tests run that may not uncover all the bugs ;) ) until we started using it in our application. Then we stated getting exceptions all over the place because we couldn't create an instance of a class because its' initializer had blown up. Therefore, we had to go away from **Abort()**. In my eyes, that call is fairly dangerous and really should be avoided - there are better, safer ways to cancel work in another thread.

> Published: 11.14.2004 11:24:59 PM CST