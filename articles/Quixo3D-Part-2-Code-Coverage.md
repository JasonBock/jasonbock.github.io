---
title: Quixo3D - Part 2: Code Coverage
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Quixo3D - Part 2: Code Coverage

[Last time we met](https://jasonbock.net/articles/Quixo3D-Part-1-Code-Coverage.html), I was talking about Code Analysis for the Quixo3D framework and engine assemblies. Today I'm going to talk about Code Coverage.

As I type, my current coverage numbers are:

* Quixo3D.Framework - 94.91%
* Quixo3D.Engine - 95.83%

I'm really trying to get as close as I can to 100% as I possibly can, just to see what I can learn from the experience. So far, I've learned a couple of things.

First, reviewing the results has helped tremendously in eliminating "unused" code. There were sections in my code base that I wasn't touching in my tests, and I quickly realized I would **never** touch it. Basically, it was unnecessary code (e.g. private no-argument constructors that I foolishly added), so I yanked that stuff out. Second, it helped me write useful utilities for testing. I have a couple of custom exception classes, and originally I had no tests around them. To ensure that they would work under serialization/`AppDomain` boundaries, I wrote a base class that would test any exception and ensure it followed the basic exception design principles: (i.e. it's serializable, it has a number of "standard" constructors, etc.). Next, it's showing me where I'm still missing coverage. I have a custom formatter to serialize and deserialize a board, but I don't have coverage over some of the property getters and setters, so I have to fix that. Finally, I have a class called `ValidMove` that handles `IEquatable<T>`, the "==" and "!=" operators, etc. but because its constructor is `internal`, I can't create the scenario where I'll get two `ValidMoves` back from the board. In fact I should never get that back. So I think I can write some tests that execute all of the "equals" and "not equals" cases, but I haven't done that just yet.

All that said, there's some areas of concern. One is enumerators. C# generates some classes when you do the `GetEnumerator()` duck typing pattern, and even though all of the code shows up being covered, the reports shows "you're close but not 100%". I'm not really sure if this is a bug or a feature. Another issue is when I create an internal class that implements `IEnumerable<T>`. I have to implement the non-generic `GetEnumerator()` method, but I'll never call it in my code. So ... it's going to sit unused. I could go the `PrivateObject` approach or other Reflection-based techniques to "cover" that code, but I think that's useless. I try to keep it black-box in my unit tests; it would feel downright silly to write a test just to have this method called.

My general conclusion is that you should strive for at least 90% coverage in your code. After that, the ROI of achieving a higher number has to be kept in mind. You may be able to get around 95% with little effort, but I think that there may be cases where 100% just won't be achievable, or, to hit that number requires techniques that I think are suspect for unit testing. I think that's OK. The more code coverage you have, the better off you'll be in the long run; just be realistic with what you can (and should) do in your unit tests.

Next up ... desiging a smart engine ...

> Published: 11.09.2007 09:27:01 AM CST