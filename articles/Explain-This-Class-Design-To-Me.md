---
title: Explain This Class Design To Me
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Explain This Class Design To Me

Take a look at `System.Reflection.Emit.ILGenerator`. Notice that it's **not** `sealed`, so you can use it as a base class. It's also `public`, so I have access to it as well.

Now take a look at the constructors.

Oh, you can't, because they're `internal` or `private`.

Why oh why oh why oh why ...

I felt so good for a second or two, and now I just feel like `ILGenerator` is pointing at me and laughing.

> Published: 10.12.2006 01:59:17 PM CST