---
title: Refining Dynamic
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Refining Dynamic

I just had a thought this morning about the `dynamic` keyword in C# 4.0. Right now, this is how you use it:

```c#
dynamic x = new Something();
x.Configure();
x.Reconfigure();
x.Allocate();
```

I wonder if it could be refined a bit to allow something like this:

```c#
var x = new Something();
x.Configure();
dynamic x.Reconfigure();
x.Allocate();
```

That is, you don't have to have the object "being dynamic" the entire time. Only the call to `Reconfigure()` is done using the underneath-the-scenes `CallSite` magic that C# 4.0 does to make a dynamic call.

I realize that you can solve this by doing some casts:

```c#
var x = new Something();
x.Configure();
((dynamic)x).Reconfigure();
x.Allocate();
```

But...this feels kind of yucky.

I'm guessing that in most scenarios a developer wants the object to be dynamic for all of the calls that occur after `dynamic` is used. But by having the keyword available on method calls as well, this gives a bit more flexibility. At the end of the day, however, I doubt this will considered to be a needed feature for C# developers. Still, I wanted to throw it out there.

> Published: 12.22.2008 10:50:55 AM CST