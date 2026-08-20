---
title: ICloneable<T>?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `ICloneable<T>`?

In 2.0 generics came into the picture, and they started to permeate the .NET world in a big way. The easiest change was to the collection classes, but there were also some not-so subtle additions and changes. One I really like is `IEquatable<T>`, which gives you a nice type-safe `Equals` method (and makes overriding `Object.Equals()` really easy). `IEnumerable` was also genericized. But ... why isn't there an `ICloneable<T>`?

I did some searching, and it seems like `ICloneable` isn't viewed in a positive light with the MS folks. Given this view, it makes more sense why that interface wasn't added in 2.0. It's not that hard to make:

```c#
public interface ICloneable<T>
  where T : class
{
  T Clone();
}
```

This doesn't inherit from `ICloneable`, but that's OK as other generic interfaces don't descend from their non-generic counterparts. Also, it doesn't work for value types.

Until a "final" judgement comes out on `ICloneable` it looks like you'll need to make your own type-safe cloning interface if you really want it. But it seems like people generally avoid `ICloneable`. I'm not a lover or a hater of it; I just found it a bit odd that a genericized version wasn't made available in 2.0, that's all.

> Published: 07.11.2007 10:16:23 PM CST