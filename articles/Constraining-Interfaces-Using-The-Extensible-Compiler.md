---
title: Constraining Interfaces Using the Extensible Compiler
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Constraining Interfaces Using the Extensible Compiler

A while back I talked about the idea of constraining an interface such that types that implemented that interface had to be of a certain type. I've added support for this idea for the Extensible Compiler for .NET via metadata (note that this drop is version 0.2.0.1, which has some minor bug fixes):

```c#
[Constrained(typeof(GoodConstrainedInterfaces))]
public interface IGoodConstraint {}

public class GoodConstrainedInterfaces : IGoodConstraint 
{
  void AMethod();
}

[Constrained(typeof(Uri))]
public interface IBadConstraint {}

public class BadConstrainedInterfaces : IBadConstraint {}
```

As you can guess by the names, `GoodConstrainedInterfaces` is OK, but `BadConstrainedInterfaces` is not. Unfortunately, this doesn't provide the assumptions that I mentioned here for variables of a constrained type. In other words, my addition doesn't make the following code valid:

```c#
public void UseConstrained(IGoodConstraint x)
{
  x.AMethod();
}
```

But you can now do this knowing that extended compiler would ensure the safety of the cast (so long as `x` isn't null!):

```
public void UseConstrained(IGoodConstraint x)
{
  (x as GoodConstrainedInterfaces).AMethod();
}
```

I'm getting to like this extended compiler idea. It's not extremely powerful, but it's definitely adding a dimension to my coding as I can extend my code via compile-time attributes.

> Published: 03.30.2005 03:36:40 PM CST