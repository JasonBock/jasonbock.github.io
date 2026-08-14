---
title: Yet Another Reason I Use Code Analysis in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Yet Another Reason I Use Code Analysis in .NET

I'm doing some work on [Spackle](https://www.nuget.org/packages/Spackle) to pass the time (have to keep those coding skillz alive), and today I'm adding a `Shuffle()` extension method to `IList<T>`-based objects. I'm using the Fisher–Yates shuffle, as implemented by Durstenfeld (see [this page](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle) for details). Anyway, I have two versions of this method: one that takes the list, and one that also takes a `SecureRandom`-based object. That signature looks like this:

```c#
public static void Shuffle<T>(this IList<T> @this, SecureRandom random)
```

I always have Code Analysis turned on for every project I work on. When I compiled to code, I got this error:

> Error 1 CA1011 : Microsoft.Design : Consider changing the type of parameter 'random' in 'IListOfTExtensions.Shuffle<T>(this IList<T>, SecureRandom)' from 'SecureRandom' to its base type 'Random'. This method appears to only require base class members in its implementation. Suppress this violation if there is a compelling reason to require the more derived type in the method signature.

I designed `SecureRandom` to use `Random` as its base class, which is why I'm getting this error. In this case, I should allow a user to provide a `Random`-based class if they really want to use a pseudo-random number generator (as you can do the same thing with `SecureRandom` if you provide a bad implementation of `RandomNumberGenerator` with one of its constructors).

This is cool, because it never dawned on me to use the base class in the method signature. And this is why I like using automated code review tools like Code Analysis because they can remind me of things like this.

> Published: 08.20.2009 10:49:25 AM CST