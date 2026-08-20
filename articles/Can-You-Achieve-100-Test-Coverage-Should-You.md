---
title: Can You Achieve 100% Test Coverage? Should You??
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Can You Achieve 100% Test Coverage? Should You??

Short response is, it depends.

Here's the longer version. At the last Code Camp I got into some discussions about test-driven development and code coverage. As an exercise in experience/frustration/observation, I was trying to see what it would take to hit 100% coverage on my tests for my Quixo3D.Framework assembly (I'll be talking about that code in the very near future), and I realized I'll never get it, unless I'm willing to use Reflection in a test, and even then, the benefit of running such a test seems odd.

Here's a brief snippet of what I'm talking about:

```c#
internal sealed class MyStuffCollection : IEnumerable<MyStuff>
{
  IEnumerator IEnumerable.GetEnumerator()
  {
    throw new NotImplementedException();
  }
}
```

This code won't compile, but what I'm trying to get across is that I have a class that only used internally, and since it implements `IEnumerable<>`, I have to implement the non-generic `GetEnumerator()`. I'll never use this method internally within the framework, and a user of the API will never see this method. But because I'm forced to implement it, it ends up being dead code. Sure, I could use some Reflection-based techniques to hit that method, but what's the point?

What I'm trying to get across is, if someone says you must hit 100% coverage in tests, that may be an invalid request. Granted, I love looking at the code highlighting in VS and seeing what code hasn't been hit yet, because it means I might be able to eliminate some code, or I'm missing some testing opportunities. But there is a cutoff where writing more tests to eek out another 0.5 percentage points is just not worth the investment in time.

> Published: 10.29.2007 09:36:56 AM CST