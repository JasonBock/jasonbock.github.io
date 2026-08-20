---
title: Reverse Code Analysis in Visual Studio
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reverse Code Analysis in Visual Studio

I just realized something tonight that's kind of annoying about Code Analysis in VS. If you have it on by default to give error rather than warnings, and you're creating new code, you may get a flurry of messages like "CA1811:AvoidUncalledPrivateCode", because your private method doesn't have any callers ... yet. So you suppress the message and now your code has an attribute on the method. But you know you're going to have an upstream caller soon!

What would be nice is if Code Analysis could do a reverse analysis. Meaning, not only does it look at your code for errors, it also looks for `SuppressMessageAttributes` and sees if your code still needs that suppression. If not, it automatically removes it for you. I think that would be a **killer** feature of Code Analysis (unless it has this in 2005 or it's getting it in 2008, but from what I can tell this feature doesn't exist).

> Published: 10.08.2007 09:09:34 PM CST