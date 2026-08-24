---
title: The Myth of `IDisposable`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Myth of `IDisposable`

A while back at the client I was asked to do some COM interop through a class they called `ComObject`. Basically, it was a way to wrap a COM reference in an object field and call methods and properties on it via a loose, `IDispatch`-style invocation. I wasn't too thrilled with this approach, but I voiced my opinion and moved on. One thing that did bother me that I couldn't let go, though, was the fact that `ComObject` didn't implement `IDisposable`. This concerned me, because `Marshal.ReleaseComObject()` was never called on the COM server. I mentioned this to my boss, and his response surpised me. He thought it was a good idea, but he thought that the runtime would automatically call `Dispose()` when the reference went out of scope.

Yikes!

I thought this myth was laid to rest years ago, and here people still think `IDisposable` is some magical interface that the runtime treats in a special fashion. So, let me state my mind so I can have some peace:

> The CLR doesn't give a crap about `IDisposable`.

You have to call it yourself. That's why using was overloaded in C# to make it easier for developers to use `IDisposable`-based objects correctly. But you still have to create the disposable object in a using block; otherwise, you lose all the benefits of early resource cleanup in that disposable reference.

OK, that is all - time to enjoy the rest of my Sunday :)

> Published: 11.20.2005 05:18:15 PM CST