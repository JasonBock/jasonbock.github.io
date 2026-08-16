---
title: One of the Oddest APIs I've Encountered in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# One of the Oddest APIs I've Encountered in .NET

It's been there forever, so I wonder how I missed this until now:

```c#
var stream = Console.Out;
Console.SetOut(new StringWriter());
```

I have to use a property to get the value, but a method to set the value?

Yikes!

There has to be a sane reason why this was done this way. I mean, why not make the property writeable?

> Published: 11.17.2008 09:07:54 PM CST