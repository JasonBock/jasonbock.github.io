---
title: When to Use InternalsVisibleTo
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When to Use InternalsVisibleTo

There's an attribute in the .NET framework called `InternalsVisibleToAttribute`, which basically allows a developer to let an assembly see the non-public members of another assembly. So if I wrote the following class in the assembly `PrivateStuff`:

```c#
using System;

namespace PrivateStuff
{
  internal sealed class YouCantSeeMe
  {
    internal void FindIt()
    {
    }
  }
}
```

Then this class in `PrivateStuff.Client` can't see it:

```c#
using PrivateStuff;
using System;

namespace PrivateStuff.Client
{
  public sealed class LetMeSeeYou
  {
    public void UseIt()
    {
      // The following two lines of code won't work.
      YouCantSeeMe haha = new YouCantSeeMe();    
      haha.FindIt();    
    }
  }
}
```

But if I add the following attribute:

```c#
[assembly: InternalsVisibleTo("PrivateStuff.Client")]
```

Then the code in `PrivateStuff.Client` will compile and work just fine.

> Note that this attribute only allows you see the internal types. If I had `FindIt()` as `private`, `LetMeSeeYou` wouldn't be able to call it.

When I first found out about this attribute (which was when 2.0 was showing up), I initially had some skittish feelings about it, but now that I've actually used it, I find that it has its place in certain places. Here's two scenarios that come to mind.

* *Test assemblies*. If you have an assembly called `Core`, and you have all the tests for `Core` in `Core.Tests`, you can add `InternalsVisibleTo` in `Core` so you can cover more code in `Core.Tests`. There is a danger of abuse here - I find that doing black-box testing is the way to go, and if tests are diving into non-public members then something feels wrong. However, there may be times where you want a test assembly to have easier access to internal types, and `InternalsVisibleTo` makes that possible.
* *The "it's public but you shouldn't touch me scenario" comment*. Sometimes I've run across the following verbage in the .NET docs: "and is not intended to be used directly from your code" (look up `ProcessProtocolHandler` in the 3.5 docs for an example). If you really don't want anyone to create it, then why make it `public`? That seems like a flaw in the design of the API. A better choice would be to use an interface and have the type implement that interface but have it `internal`. Or, if another trusted assembly needs that type, just add `InternalsVisibleTo` to the target assembly.

By the way, remember that if you don't strong-name your assemblies, anyone can easily create a client assembly with the right name and have easy access to non-public types in an assembly that contains `InternalsVisibleTo`. However, if you're not producing assemblies for purchases, this is usually not a big deal. Also, even if you do strong-name assemblies such that you're only allowing clients with a specific strong-name to have access to non-public types, anyone with knowledge of Reflection can use non-public members (as long as they have the permission to do so).

> Published: 01.03.2008 07:57:04 AM CST