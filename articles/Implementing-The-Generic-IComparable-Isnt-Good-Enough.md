---
title: Implementing the Generic IComparable Isn't Good Enough
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Implementing the Generic `IComparable` Isn't Good Enough

I'm sure this is common knowledge by now, but I just got bit by it. If you implement `IComparable<T>`, that may not be enough. I was doing a custom search on an array of objects that implemented `IComparable<T>`, but I kept getting an error that said I had to implement `IComparable`. This really threw me, but as soon as I implemented `IComparable` (and delegated its work to the generic version), things worked as expected.

> Published: 10.29.2007 01:15:32 PM CST