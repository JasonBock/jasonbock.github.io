---
title: When Using Isn't Enough For Code Analysis ... And When You Don't Need It (?)
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When Using Isn't Enough For Code Analysis ... And When You Don't Need It (?)

Consider the following code:

```c#
using System;
using System.Threading;
using ST = System.Timers;

namespace TrickyDisposing
{
  public sealed class TimerUsage
  {
    private ST.Timer timer = new ST.Timer();

    private void DoIt()
    {
      this.timer.Start();
    }

    public void TimeIt()
    {
      using(EventWaitHandle handle = new ManualResetEvent(false))
      {
        ST.ElapsedEventHandler handler = null;

        try
        {
          handler = (sender, e) =>
          {
            this.timer.Stop();
            handle.Set();
          };

          this.timer.Interval = (new Random()).NextDouble() * 1000;
          this.timer.Elapsed += handler;

          this.DoIt();
          handle.WaitOne(10000, true);
        }
        finally
        {
          if(handler != null)
          {
            this.timer.Elapsed -= handler;
          }
        }
      }
    }
  }
}
```

Now, `System.Timers.Timer` implements `IDisposable`, so there's a problem, because `TimerUsage` doesn't implement `IDisposable`. But what's really odd is the error message you get from Code Analysis:

```
Warning 3 CA1001 : Microsoft.Design : Implement IDisposable on 'TimerUsage' because it creates members of the following IDisposable types: 'ManualResetEvent', 'Timer'. If 'TimerUsage' has previously shipped, adding new members that implement IDisposable to this type is considered a breaking change to existing consumers.
```

Huh? The handle variable is in a `using` statement, so why does the error message say that it needs to be disposed?

The issue is with how lambda expressions (or anonymous methods) are implemented in C# and how Code Analysis interprets it. Here's a version of the class that is exactly the same, but shows all of the details:

```c#
using System;
using System.Runtime.CompilerServices;
using System.Threading;
using ST = System.Timers;

namespace TrickyDisposing
{
  public sealed class ReflectorTimerUsage
  {
    private ST.Timer timer = new ST.Timer();

    private void DoIt()
    {
      this.timer.Start();
    }

    public void TimeIt()
    {
      ST.ElapsedEventHandler outerHandler = null;
      AnonymousMethod method = new AnonymousMethod();
      method.capturedThis = this;
      method.handle = new ManualResetEvent(false);

      try
      {
        ST.ElapsedEventHandler handler = null;
        try
        {
          if(outerHandler == null)
          {
            outerHandler = new ST.ElapsedEventHandler(method.Invoke);
          }

          handler = outerHandler;
          this.timer.Interval = new Random().NextDouble() * 1000.0;
          this.timer.Elapsed += handler;
          this.DoIt();
          method.handle.WaitOne(10000, true);
        }
        finally
        {
          if(handler != null)
          {
            this.timer.Elapsed -= handler;
          }
        }
      }
      finally
      {
        if(method.handle != null)
        {
          ((IDisposable)method.handle).Dispose();
        }
      }
    }

    [CompilerGenerated]
    private sealed class AnonymousMethod
    {
      public ReflectorTimerUsage capturedThis;
      public EventWaitHandle handle;

      public void Invoke(object sender, ST.ElapsedEventArgs e)
      {
        this.capturedThis.timer.Stop();
        this.handle.Set();
      }
    }
  }
}
```

Note that I used Reflector to generate this code. I had to tell it to show code using C# 1.0 syntax, and I also cleaned up some of the class names (C# 3.0 will create some nasty mangled names that you couldn't use as names for classes, but they're fine at the IL level).

Here's the issue. The nested class `AnonymousMethod` captures all of the information it needs to implement the lambda expression (the `Invoke()` method). In this case, it has to capture two variables, the "this" value (to reference the timer) and the handle. Notice that it doesn't capture the timer itself; it gets a reference to the object. This means that the `AnonymousMethod` object has a reference to a timer via the `TimerUsage` instance, and that timer also has a reference the `AnonymousMethod` instance via the `Elapsed` event.

That's not good. It's OK to have these circular reference by Code Analysis standards if the containing class implements `IDisposable` [1]:

```c#
using System;
using System.Threading;
using ST = System.Timers;

namespace TrickyDisposing
{
  public sealed class TimerUsageWithDispose : IDisposable
  {
    private ST.Timer timer = new ST.Timer();

    public void Dispose()
    {
      this.timer.Dispose();
    }
        
    private void DoIt()
    {
      this.timer.Start();
    }

    public void TimeIt()
    {
      using(EventWaitHandle handle = new ManualResetEvent(false))
      {
        ST.ElapsedEventHandler handler = null;

        try
        {
          handler = (sender, e) =>
          {
            this.timer.Stop();
            handle.Set();
          };

          this.timer.Interval = (new Random()).NextDouble() * 1000;
          this.timer.Elapsed += handler;

          this.DoIt();
          handle.WaitOne(10000, true);
        }
        finally
        {
          if(handler != null)
          {
            this.timer.Elapsed -= handler;
          }
        }
      }
    }
  }
}
```

In this case, Reflector generates this:

```c#
using System;
using System.Runtime.CompilerServices;
using System.Threading;
using ST = System.Timers;

public sealed class TimerUsageWithDispose : IDisposable
{
  private Timer timer = new Timer();

  public void Dispose()
  {
    this.timer.Dispose();
  }

  private void DoIt()
  {
    this.timer.Start();
  }

  public void TimeIt()
  {
    ElapsedEventHandler outerHandle = null;
    AnonymousMethod method = new AnonymousMethod();
    method.capturedThis = this;
    method.handle = new ManualResetEvent(false);
        
    try
    {
      ElapsedEventHandler handler = null;
            
      try
      {
        if (outerHandle == null)
        {
          outerHandle = new ElapsedEventHandler(method.Invoke);
        }
        handler = outerHandle;
        this.timer.Interval = new Random().NextDouble() * 1000.0;
        this.timer.Elapsed += handler;
        this.DoIt();
        method.handle.WaitOne(10000, true);
      }
      finally
      {
        if (handler != null)
        {
          this.timer.Elapsed -= handler;
        }
      }
    }
    finally
    {
      if (method.handle != null)
      {
        ((IDisposable) method.handle).Dispose();
      }
    }
  }

  [CompilerGenerated]
  private sealed class AnonymousMethod
  {
    public TimerUsageWithDispose capturedThis;
    public EventWaitHandle handle;

    public void Invoke(object sender, ElapsedEventArgs e)
    {
      this.capturedThis.timer.Stop();
      this.handle.Set();
    }
  }
}
```

You can see even though we get this circular reference set up again, Code Analysis doesn't bark at us.

Of course, you can solve the problem by not having the timer as a field, but as a local variable:

```c#
using System;
using System.Threading;
using ST = System.Timers;

namespace TrickyDisposing
{
  public sealed class BetterTimerUsage
  {
    private void DoIt(ST.Timer timer)
    {
      timer.Start();
    }

    public void TimeIt()
    {
      ST.Timer timer = new ST.Timer();

      using(EventWaitHandle handle = new ManualResetEvent(false))
      {
        ST.ElapsedEventHandler handler = null;

        try
        {
          handler = (sender, e) =>
          {
            timer.Stop();
            handle.Set();
          };

          timer.Interval = (new Random()).NextDouble() * 1000;
          timer.Elapsed += handler;

          this.DoIt(timer);
          handle.WaitOne(10000, true);
        }
        finally
        {
          if(handler != null)
          {
            timer.Elapsed -= handler;
          }
        }
      }
    }
  }
}
```

What I find really funny in this case is Code Analysis doesn't bark at for not putting the timer in a `using` statement!

So, the moral of the story is, the messages you get from Code Analysis may not always make sense at first glance because of what C# is doing underneath the scenes. And sometimes ... you gotta wonder if it's missing some code issues.

[1] Don't implement `Dispose()` this way - this is a crude incomplete implementation just to illustrate a point. Please refer to the SDK to see how it should be done.

> Published: 03.12.2008 07:46:21 PM CST