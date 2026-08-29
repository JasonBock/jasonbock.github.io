---
title: TypeMock
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# TypeMock

I'm not a big user of mock objects, as I generally think that you should always test your code against the "real thing." But this is a rule of thumb and not an absolute (which seems to be the case for a lot of things in life). Anyway, on the internal Magenic lists someone was talking about nMock, but they quickly noticed that it does dynamic proxies so methods much virtual, the class can't be sealed, etc. Then someone mentioned TypeMock. What's cool about that tool is it uses the profiling API to inject mock implementations. Wow, that's really cool!

As I haven't used either product I have no opinion on their usage, bugs, etc. I just found the implementation details to TypeMock quite intriguing.

> Published: 05.12.2005 03:18:57 PM CST