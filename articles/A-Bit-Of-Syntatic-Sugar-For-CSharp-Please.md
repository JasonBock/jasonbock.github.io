---
title: A Bit of Syntatic Sugar for C# Please
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Bit of Syntatic Sugar for C# Please

When I reference a method in C#, I always qualify it with its point of origin. Meaning if I call an instance method, I do this:

```c#
this.CallMe();
```

A static method is declared this way:

```c#
BigDescriptiveClassName.CallMe();
```

Now, I know that some people would say this is excessive but I think this makes the code more readable. But anyway ... here's my point.

I'd love to be able to declare the static method this way:

```c#
class.CallMe();
```

There's a keyword for the instance reference ... can I have a keyword for the class as well? "class", "static", whatever, I don't really care, just something that takes less keystrokes.

kthxbye :)

> Published: 11.03.2008 08:38:30 AM CST