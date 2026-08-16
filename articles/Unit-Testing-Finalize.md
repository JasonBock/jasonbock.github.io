---
title: Unit Testing Finalize
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unit Testing Finalize

Throwing out a philosophical question here:

Should unit tests try to call `Finalize()`?

Let's say I have a disposable object. Following the coding pattern for `IDisposable`, I should create a finalizer, and call a private version of `Dispose()` that takes a bool, which would take `false` in the finalizer. Now, finalizers are called by the garbage collector, so trying to do something in the test where I'd call `GC.Collect()` a number of times feels ugly. I could go the Reflection route, find the method, and call it ... this seems like a "cleaner" approach, but is this overkill? But it is a method I've declared in my class and not having code coverage around it doesn't seem right either.

I really don't have any solid views either way on this ... feel free to comment with your thoughts, comments, emotional outbreaks, etc.

> Published: 11.03.2008 09:55:03 AM CST