---
title: TryParse Inconsistencies
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# TryParse Inconsistencies

One thing I like in .NET 2.0 is the addition of a method called TryParse() on a number of classes (usually value types) ([here's a link](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/exceptions-and-performance#try-parse-pattern) to the "TryParse pattern" entry in MSDN help). Examples of this pattern are `Int32.TryParse()`, `IPAddress.TryParse()`, and `DateTime.TryParse()`. I like `TryParse()` because it eliminates the need to add an exception handler in my code. However, it's absent from other classes - the biggest miss is on the `Guid` class. I wonder why it was added to some classes and not others? Of course, it's a static method so they couldn't create an `IParseable<T>` interface with a `TryParse()` method on it. Plus, different classes need different definitions of `TryParse()` - e.g. `DateTime.TryParse(String, IFormatProvider, DateTimeStyles, DateTime)` vs. `Double.TryParse(String, NumberStyles, IFormatProvider, Double)`. But I still find it odd that it's missing on some classes. Hopefully it'll be added in 3.0 ...

> Published: 04.26.2006 07:58:57 PM CST