---
title: The Road to VB7
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Road to VB7

## Introduction

Today marks what I consider to be one of the biggest days in VB's history. At VBITS, Steve Ballmer gave a keynote address that gave us a revealing glimse as to what will be in VB7. In this informal article, I'll state my own opinion on what lies ahead on the long path to VB7.

## The Pleading is Over ... Probably

If you haven't checked out the new features in VB7, click here. I won't cover all of them in detail, because the article does a good job of summing up the main features (although I will comment on some of the features in a moment). But it goes without saying that the wait for the significant advances in VB may be over. Free-threading, structured exception handling, true OOP, type safety (**Option Strict!!**) ... the list goes on. True, the C++, Java, and an assortment of other languages have had most or all of these features for a long time - I won't argue that point. But at least there's a light at the end of the tunnel, and, although history has shown that the light from Microsoft is more often than not a train, it probably isn't this time. I'm really happy that MS hasn't done the "we-got-some-really-cool-stuff-in-the-next-version-just-wait-and-see" approach this time. True, NDAs will prevent those on the beta test team from spilling their guts on what's really going on, but the information is fairly open as I write this.

## Some General Comments

It's probably way too early to discuss the new features coming in VB7 (especially basing my observations on code that I didn't see compiled), but I do have some quick observations on MS's VB Language Innovations article:

* I'm glad that we'll have structured exception handling, but I hope they beef up the `Catch` block. That way, you could catch specific errors in different blocks. Yes, it's Javaish, but I think it makes sense.
* `Option StrictAndIReallyMeanIt` is finally here (as `Option Strict` ;) ). No more evil type-coercion.
* Yes, safe free-threading is here! The days of `CreateThread` are over! A couple of things I noticed on their threading example. One is that it seems like `AddressOf` will work for class module methods. Also, the method that is run on another thread isn't limited to a specific name. That's pretty cool, although I hope they add a `Variant` parameter so you can pass information into the thread. Of course, it seems like you'll be able to write thread-neutral apartment objects in VB7, so you can create and initialize the object in one thread (yes, we'll have constructors and it doesn't look like the `IObjectConstruct` COM+ "construction."), and have the new thread read the initialization information. But I'm still hoping that you can pass in a parameter.
* Yes, we'll have a true OOP language in VB7 (inheritance, constructors, and method overloading, oh my!). Those who program in VB7 won't have to hang their heads in shame when they're in the OO language crowds. I'm curious as to how VB7 will do this (implementation inheritance isn't allowed in COM as far as I know) - my guess is that VB classes are separate from COM coclasses.

## The Road Ahead

Already newsgroups are buzzing about VB7 and its' new features, and we still have a year to wait. So it's safe to say that my initial thoughts on this information may be completely off-base. But I will say this: I was personally thinking of moving away from VB because I was getting sick of its' limitations. I got certified in Java, and I was starting to take a serious look at Eiffel (an endeavour that I'll still do). But after reading what came out today, I have to say I may stick around for a while. I agree with Patrick Meader's musings in that VB7 will be a major jump for a fair amount of VB programmers who have hacked their way through projects. Upgrading from pre-VB7 applications will probably not be as easy, since Patrick notes that VB7 is a complete rewrite. But there comes a time where you have to say, "We've taken this as far as it can go - it's time to move on." I'd rather have MS take their punches and remove backwards compatiabilities in some areas if, in the long run, it helps us out. And it looks like VB7 will be a major improvement. When Bruce McKinney left VB, he stated, "I hope it achieves its potential in future versions." It looks like that day is one year away.

## The Future of Programming In General

Ultimately, I'll develop systems in most any tool or language. When it comes down to it, the language is starting to lose its' importance. Granted, it's great to see VB get the overhaul so many of us have been desperately asking for. But let's face it, languages are more similiar than difference. Come to think of it, the OS doesn't really matter anymore either. What matters is interoperability and connectedness. SOAP helps out with this tremendously. In the near future, the language wars and OS battles will be a thing of the past (at least I hope so). You won't get rid of Microsoft. Linux isn't going away. Java is here to stay. Let's deal with this. Don't force someone to using tool X that uses language Y on OS Z. Just make sure we can all communicate effectively. With SOAP, we're on our way ... finally.

## Addendums

### 12.05.2000

As we all know by now, a lot has happened in a year. .NET became the thing to talk about and know before anyone else. VB7 is now VB.NET. .NET supports a ton of languages. .NET is just a Java rip-off ;). Personally, I think all of this has been a good thing (although some of the changes from VB6 to VB.NET just don't make any sense to me). But time will tell how this will all pan out. Ultimately, I think C# is the language of choice. I've never liked languages that force you to use a semicolon, but I think C# gives you the best balance out of all of the available languages for writing .NET applications. We'll see, though. We'll see ...

> Published: 02.15.2000 12:00:00 AM CST