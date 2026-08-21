---
title: It's C#, Not C++
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# It's C#, Not C++

You know you're working with a former/current C++ developer when you see this in C# code:

```c#
// Assume Data has been defined somewhere...
Data data = /* get string value */;

if(null != data)
{
 // ...
}
```

I know why they're doing this, but it's not necessary in C#.

> Published: 05.04.2007 08:55:35 AM CST