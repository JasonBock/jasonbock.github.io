---
title: CA2000 Is No Longer in VS2008
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# CA2000 Is No Longer in VS2008

I'm putting material together for my "Writing Better Code" talk, and it was interesting to find out that the CA2000 rule (which detects if you don't call `Dispose()` on a `IDisposable`-based object) was removed in VS2008. There's a post that has a spreadsheet that shows it being removed, and there's another post that talks more about why this rule was pulled. What's interesting about that first post is that they're talking about using Phoenix in a future version of the Code Analysis tools. That would suggest (at least to me it does) that Phoenix is going to get a lot more exposure in the .NET world in the (near?) future.

> Published: 03.16.2008 10:38:56 AM CST