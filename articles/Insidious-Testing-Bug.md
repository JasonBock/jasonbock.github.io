---
title: Insidious Testing Bug
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Insidious Testing Bug

Hold on to your hats, because explaining this one is going to take a bit of time/code. Hopefully this will help out others who have used anonymous delegates to handle events in their test methods. I've distilled the problem down to its essence - I ran into this doing a bunch of WCF extensibility stuff so the real code is much more involved that what I'll show, but the problem is still the same (and in fact, the problem has nothing to do with WCF).

To start, here's the test class:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.Threading;

namespace UnregisteringAnonymousEventHandlers
{
  [TestClass]
  public class RaisingEventsTests
  {
    [TestMethod]
    public void Go1()
    {
      using(EventWaitHandle handle = new ManualResetEvent(false))
      {
        bool wasSet = false;
                
        RaisingEvents.RaiseIt += new EventHandler(
          delegate
          {
            handle.Set();
            wasSet = true;
          });
                
        RaisingEvents raising = new RaisingEvents();
        raising.Go();
                
        Assert.IsTrue(handle.WaitOne(20000, true));
        Assert.IsTrue(wasSet);
      }
    }

    [TestMethod]
    public void Go2()
    {
      using(EventWaitHandle handle = new ManualResetEvent(false))
      {
        bool wasSet = false;

        RaisingEvents.RaiseIt += new EventHandler(
          delegate
          {
            handle.Set();
            wasSet = true;
          });

        RaisingEvents raising = new RaisingEvents();
        raising.Go();

        Assert.IsTrue(handle.WaitOne(20000, true));
        Assert.IsTrue(wasSet);
      }
    }
  }
}
```

`RaisingEvents` looks like this:

```c#
using System;

namespace UnregisteringAnonymousEventHandlers
{
  internal sealed class RaisingEvents
  {
    internal static EventHandler RaiseIt;
        
    public void Go()
    {
      if(RaisingEvents.RaiseIt != null)
      {
        RaisingEvents.RaiseIt(this, EventArgs.Empty);
      }
    }
  }
}
```

So, the testing code is pretty straightforward. All I'm doing in my unit tests is making sure that `RaiseIt` is raised when `Go()` is called. But problems start to occur during test runs. If I ran `Go1()` or `Go2()` by themselves, the test run was successful. However, if ran them together, then I got the following error:

```
Test method UnregisteringAnonymousEventHandlers.RaisingEventsTests.Go2 threw exception: System.ObjectDisposedException: Safe handle has been closed.
```

This message is actually very helpful, once you untangle the mess I've made in my testing code. Here's what happens. `Go1()` is invoked first. An anonymous method is attached to handle `RaiseIt`. `Go()` is called, and the asserts succeed. Now, `handle` is wrapped in a `using` block, so `handle` is disposed before `Go1()` completes (I bet some of you already know where this is going). Now, the same thing happens in `Go2()`, but when `Go()` is called, the event handler thinks there are 2 methods to call, not just one. Why? Well, in `Go1()`, **I never unattached the event handler**. So this time around, the anonymous method in `Go1()` will be invoked, but handle has already been disposed, and kablooey!

The way to fix this is to hang on to a reference to the anonymous delegate and unattach the event handler - I'll just show the change in one of the test methods:

```c#
[TestMethod]
public void Go1()
{
  using(EventWaitHandle handle = new ManualResetEvent(false))
  {
    bool wasSet = false;

    EventHandler handler = new EventHandler(
      delegate
      {
        handle.Set();
        wasSet = true;
      });

    RaisingEvents.RaiseIt += handler;
        
    RaisingEvents raising = new RaisingEvents();
    raising.Go();
        
    Assert.IsTrue(handle.WaitOne(20000, true));
    Assert.IsTrue(wasSet);

    RaisingEvents.RaiseIt -= handler;     
  }
}
```

Now both tests will succeed, no matter if they're executed individually or together.

As a side note, it would be nice if the `using` statement could be extended to allow developers to hook into the "finally" aspect:

```c#
// Note: This is INVALID C# code - endusing doesn't exist!
[TestMethod]
public void Go1()
{
  EventHandler handler = null;

  using(EventWaitHandle handle = new ManualResetEvent(false))
  {
    bool wasSet = false;
        
    handler = new EventHandler(
      delegate
      {
        handle.Set();
        wasSet = true;
      });
            
    RaisingEvents.RaiseIt += handler;
        
    RaisingEvents raising = new RaisingEvents();
    raising.Go();
        
    Assert.IsTrue(handle.WaitOne(20000, true));
    Assert.IsTrue(wasSet);
  }
  endusing
  {
    RaisingEvents.RaiseIt -= handler;
  }
}
```

All `using` does is create a bunch of `try ... finally` blocks to ensure (as best as it can) that `Dispose()` is called on the "used" object. However, it would be nice (especially in this case) to allow developers to execute code that would run in the `finally` block that wraps the `Dispose()` call.

I hope this helps those who tried to be way too cute with their unit tests. Actually, I really like using anonymous delegates to handle events; I was just being a blockhead by not unattaching my events and using anonymous delegates had nothing to do with. But using anonymous delegates seemed to "hide" the problem from my eyes for some odd reason.

> Published: 02.06.2007 01:37:34 PM CST