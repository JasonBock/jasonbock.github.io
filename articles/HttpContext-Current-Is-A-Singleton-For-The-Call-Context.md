---
title: HttpContext.Current Is a Singleton for the Call Context
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `HttpContext.Current` Is a Singleton for the Call Context

I'm currently working on an abstraction for members within `HttpContext` at the client. I've tried mocking `HttpContext` in the past and it's a game of Reflection wars, one that I get tired of quick. I realize there are many options around some of the limitations present in the current version of ASP.NET, but we're basically creating a `HostContext` class that has a couple of different implementations (an ASP.NET and a test one).

OK, so here's the issue I ran into yesterday:

```c#
private static HostContext current;

public static HostContext Current
{
  get
  {
    if(HostContext.current == null)
    {
      HostContext.current = new AspNetHostContext();
    }

    return HostContext.current;
  }
  set
  {
    // ...
  }
}
```

I know some people don't like singletons - I'm not going to get into that argument right now. But here's the issue. `HttpContext.Current` **appears** to be a singleton (which is what I'm trying to model in `HostContext`), but it's not. If you dig into `HttpContext.Current`, you'll see that it's a singleton for the duration of the current call. In other words, it uses `CallContext` (more or less) to retrieve and store the current `HttpContext`.

So ... when I pushed the code above into our testing environment, things got very **bad** in a hurry. Fortunately, Reflector saved my butt (again!), because I quickly figured out what was going wrong by looking at the implementation of the `Current` property on `HttpContext`. Here's how I changed the property:

```c#
public static HostContext Current
{
  get
  {
    HostContext current = CallContext.GetData(HostContext.CallContextKey) as HostContext;

    if(current == null)
    {
      current = new AspNetHostContext();
      CallContext.SetData(HostContext.CallContextKey, current);
    }

    return current;
  }
  set
  {
    // ...
  }
}
```

Now everything's working as expected, and we can swap out the ASP.NET vs. testing context whenever we want. Nice!

> Published: 12.07.2007 10:32:04 AM CST