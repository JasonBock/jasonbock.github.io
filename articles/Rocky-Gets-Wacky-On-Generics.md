---
title: Rocky Gets Wacky on Generics
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Rocky Gets Wacky on Generics

OK, Rocky, [that's](https://web.archive.org/web/20060724144853/http://www.lhotka.net/WeBlog/PermaLink.aspx?guid=10eb26b4-f52d-4ea9-9cb8-ac8c103ee1f2) interesting (or strange - take your pick ;) ). But, the more I thought about it, I wondered what the point of this code was! Why not eliminate generics altogether and just do this:

```c#
public class BaseClass
{
  public int GetBaseAnswer() { return 42; }
}

public class StrangeClass : BaseClass
{
  public int GetStrangeAnswer() { return 123321; }
}

public class TestStrangeMethod
{
  public void Test()
  {
    StrangeMethod extendedBaseClass = new StrangeClass();
    int baseAnswer = extendedBaseClass.GetBaseAnswer();
    int strangeAnswer = extendedBaseClass.GetStrangeAnswer();
  }
}
```

I understand your points, but (and remember, it's late when I posted this) I just don't see what generics adds in terms of extensibility and flexibility to the code I give that doesn't use generics. There may be something to what you're doing in terms of the factory pattern but I'm not seeing it. [1]

By the way, my original code snippet was an attempt to create subclasses of a base class without having to physically code the subclasses in full (dynamic subclasses, if you will). I **think** this is something you can do in C++, but I'm not a C++ programmer so I'm not sure about this (I'll try to verify this).

[1] I'm sure as soon as I post this I'll "see the light". Usually when I get confused about something I have to puzzle over it and write about it, but only until I post my question do I "get it", and then I feel really stupid :).

> Published: 10.21.2004 12:11:52 AM CST