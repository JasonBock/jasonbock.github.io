---
title: Passing Methods Information
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Passing Methods Information

The title of this post may sound a little odd, but here's the situation we've run into. We're incorporating the singleton pattern in our application to manage file watching. It's actually a nice, slick design - the end result is we're doing something like this:

```c#
SpecificData data = SpecificData.Instance;
```

This works well because the file that `SpecificData` watches is named in a configuration file. But this pattern is limiting because we can't pass in the name of the file. `Instance` is a property so we can't define arguments. Even if we change it to a method like `GetInstance()`, it still won't work because the structure of our implementation requires the same signature.

There's a way around this - I'll generalize it using the creator pattern:

```c#
public static class Loader
{
  public static ILoaded Load()
  {
    return new Load();
  }
}
```

It's a simple implementation, but it shows the issue. `Load()` returns an implementation of `ILoaded`:

```c#
public interface ILoaded
{
  string Unload();
}
```

But the creational method takes no arguments. Therefore, we can't get any information to `Load`:

```c#
public sealed class Load : ILoaded
{
  private string stuff;
    
  public Load()
    : base()
  {
  }

  public string Unload()
  {
    return this.stuff;
  }
}
```

What happens if we want to get stuff set to a value, other than by reading a configuration file? There's a way to do it via `CallContext` and defining a class that implements `ILogicalThreadAffinative`:

```c#
[Serializable]
public sealed class ExtraStuff : ILogicalThreadAffinative
{
  public const string CallContextId = "ExtraStuff";

  private string stuff;

  private ExtraStuff() : base()
  {
  }

  public ExtraStuff(string stuff)
    : this()
  {
    this.stuff = stuff;
  }

  public string Stuff
  {
    get
    {
      return this.stuff;
    }
  }
}
```

We can now check to see if `CallContext` has an `ExtraStuff` instance and use that:

```c#
public sealed class Load : ILoaded
{
  private string stuff;
    
  public Load()
    : base()
  {
    object extra = CallContext.GetData(ExtraStuff.CallContextId);
        
    if(extra != null && extra is ExtraStuff)
    {
      this.stuff = (extra as ExtraStuff).Stuff;    
    }
  }

  public string Unload()
  {
    return this.stuff;
  }
}
```

Of course, the caller has to get that set for us - here's a test method to illustrate this:

```c#
private const string Stuff = "window";

[TestMethod]
public void LoadWithCallContext()
{
  CallContext.SetData(ExtraStuff.CallContextId, 
    new ExtraStuff(LoadTests.Stuff));
  Assert.AreEqual(LoadTests.Stuff, Loader.Load().Unload());
}
```

This also works in multithreaded scenarios as this test method shows:

```c#
[TestMethod]
public void LoadWithCallContextAcrossThreads()
{
  using(EventWaitHandle handle = new ManualResetEvent(false))
  {
    string result = null;
        
    CallContext.SetData(ExtraStuff.CallContextId, new ExtraStuff(LoadTests.Stuff));
    Assert.IsTrue(ThreadPool.QueueUserWorkItem(new WaitCallback(delegate
    {
      result = Loader.Load().Unload();
      handle.Set();
    })));            
        
    Assert.IsTrue(handle.WaitOne(5000, true));
    Assert.AreEqual(LoadTests.Stuff, result);
  }
}
```

I'm not saying this is an ideal solution. If you can get the information into a method via an argument, do it. But in the case where you're stuck behind a pattern, this gives you a way out.

> Published: 05.09.2007 09:17:00 PM CST