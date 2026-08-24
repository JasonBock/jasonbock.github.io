---
title: More Reasons Why Swallowing Exceptions is Not a Good Idea
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# More Reasons Why Swallowing Exceptions is Not a Good Idea

Here's an excellent blog entry on why swallowing exceptions is not a good idea. I have to admit that I've done this in the past (both far and recent), primarily in the case where a method call throws an exception and I can't crash (because it's in a WinService that has to stay up to service request, for example). However, I no longer believe this is a good practice, even in this case. Just logging the exception is not good enough. Rethrow the exception (via `throw`, **not** `throw {object reference}`!), or wrap it up as the inner exception in another exception. Of course, this is assuming that the development group has aggressive testing practices in place to catch a lot of bugs before it's released into production and that their code base is resilient enough to handle **known** exceptional cases. If you're just looking for an exception, you're trying to handle an unknown case, and that's dangerous.

> Published: 06.15.2006 08:25:54 AM CST