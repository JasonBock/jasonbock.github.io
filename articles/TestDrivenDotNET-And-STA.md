---
title: TestDriven.NET and STA
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# TestDriven.NET and STA

I think I've run into a ... well, it's not really a bug (but I wouldn't call it a feature) with TestDriven.NET. The problem is when a method that's somewhere in the call stack of the test method calls `WaitHandle.WaitAll(WaitHandle[])`. Since the add-in uses `STAThreadAttribute` on its' main method, I get a `NotSupportedException` with the following message:

```
TestCase 'xxx.xxx.QueryMultipleServers' failed: System.NotSupportedException : WaitAll for multiple handles on an STA thread is not supported.
```

It's interesting to note, thought, that the test does not fail under NUnit's GUI, even though its' `main` method is also marked with `STAThreadAttribute`. It may have something to do with the add-in running under VS .NET. It's confusing, that's for sure. If I can find a rational explanation for this I'll let you know. If you want to try it out for yourself, just use this code:

```c#
using NUnit.Framework;
using System;
using System.Threading;

namespace WaitHandlesAndSTA
{
  [TestFixture()]
  public sealed class HandleTest
  {
    [Test()]
    public void WaitForHandles()
    {
      WaitHandle[] handles = new WaitHandle[4];

      for(int i = 0; i < handles.Length; i++)
      {
        handles[i] = new AutoResetEvent(true);
      }

      WaitHandle.WaitAll(handles);
    }
  }
}
```

In NUnit (I'm using 2.2), I get a green bar. In TestDriven.NET (under VS .NET 2003), I get the exception.

**Update #1**: I just noticed that in NUnit, the `ApartmentState` property for the current `Thread` is `MTA`, and in TestDriven.NET it's `STA`. At least that explains the difference ...

**Update #2**: Seems like NUnit has a way to specify `ApartmentState`. In the constructor for `TestRunnerThread`, it checks the configuration file for different apartment state settings. For some reason I can't get the EXE's configuration file to recognize the entries that are in nunit.tests.dll.config such that I can change the apartment state that `TestRunnerThread` uses.

> Published: 02.14.2005 11:46:19 AM CST