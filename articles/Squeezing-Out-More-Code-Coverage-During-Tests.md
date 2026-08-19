---
title: Squeezing Out More Code Coverage During Tests
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Squeezing Out More Code Coverage During Tests

Recently I've been looking over some code coverage numbers in a project of mine. I noticed that one method wasn't fully covered:

```c#
public void MyMethod()
{
  this.MyPrivateMethod();
}
```

VS 2008 was showing that the call to `MyMethod()` didn't have 100% coverage. The problem was that it was being called, so I didn't understand why it didn't have 100% coverage. The issue that the one time it was being called during a test `MyPrivateMethod()` was throwing an (expected) exception. What I needed to do is write a test such that it didn't throw an exception, and then I got the 100% magic number.

My point is that even though you think you're calling a method, there may be a reason why VS is telling you that you don't have everything covered.

> Published: 01.11.2008 10:37:27 AM CST