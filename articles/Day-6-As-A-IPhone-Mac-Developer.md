---
title: Day 6 as a iPhone/Mac Developer
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Day 6 as a iPhone/Mac Developer

Today was the first day I really hosed a Mac. I was making a selector, I was in the debugger, I right-clicked on the variable, and I said something like "Open in Window". The Mac locked up, **badly**. I thought that was impossible to do on a Mac :).

I'm adding in some threading/asynch code. The problem I'm having now is calling `detachNewThreadSelector` against a target that's a class-level method. In other words, I have a method like this:

```objc
+ (void)backgroundWork:(id)data
{
   // ...
}
```

So I tried calling it like this:

```objc
CustomStuff *myData = /* initialize */;
[NSThread detachNewThreadSelector:@selector(backgroundWork:)
                         toTarget:nil
                       withObject:myData];
```

Notice that I use `nil` for `toTarget`, since `backgroundWork` is class-level. This compiles, but `backgroundWork` is never invoked. I'd like to not have create an object just to run a method on a different thread, but I can't figure this out. I ended up making the method an instance method but it would be nice to figure this problem out.

> Published: 09.17.2008 03:51:24 PM CST