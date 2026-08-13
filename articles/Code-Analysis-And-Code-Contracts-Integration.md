---
title: Code Analysis and Code Contracts Integration
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Analysis and Code Contracts Integration

I noticed something a couple of days ago. It's not a big deal, but it's a little annoying.

Let's say you have the following class:

```c#
using System;

namespace CA1062
{
  public sealed class StringLength
  {
    public StringLength(string value)
    {
      this.Length = value.Length;
    }

    public int Length { get; private set; }
  }
}
```

The point I'm illustrating here is that I'm not validating value to be non-null, which is a violation of [CA1062](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1062). If you add a null check:

```c#
using System;

namespace CA1062
{
  public sealed class StringLength
  {
    public StringLength(string value)
    {
      if(value == null)
      {
        throw new ArgumentNullException("value");
      }

      this.Length = value.Length;
    }

    public int Length { get; private set; }
  }
}
```

The rule violation goes away.

Well, now there's this Code Contracts thingee that expresses this much better (IMO):

```c#
using System;
using System.Diagnostics.Contracts;

namespace CA1062
{
  public sealed class StringLength
  {
    public StringLength(string value)
    {
      Contract.Requires<ArgumentNullException>(
        value != null, "The value argument cannot be null.");

       this.Length = value.Length;
    }

    public int Length { get; private set; }
  }
}
```

But if you do this, CA1062 shows up again. At least I couldn't find any UI configuration element in the *Code Contracts* tab in VS to get Code Analysis to be aware of this variation.

This isn't that surprising - I'm guessing the rules engine isn't Code Contracts-aware and doesn't realize this is a `null` check. It's not a big deal, but I hope the two tools can play together better in future versions.

> Published: 06.09.2010 02:57:29 PM CST