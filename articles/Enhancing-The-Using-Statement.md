---
title: Enhancing the Using Statement
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Enhancing the `using` Statement

There are two things I'd like to see added to the `using` statement. You know, it's the statement you use when you're playing with an `IDisposable`-based object:

```c#
public sealed class Disposable : IDisposable
{
  public void Go()
  {
  }	

  public void Dispose()
  {
  }
}

using(Disposable disposable = new Disposable())
{
  disposable.Go();
}
```

Essentially, C# generates code that looks something like this:

```c#
public void Use()
{
  Disposable disposable = new Disposable();

  try
  {
    disposable.Go();
  }
  finally
  {
    if (disposable != null)
    {
      disposable.Dispose();
    }
  }
}
```

One thing I'd like to see added is the ability to add exception handlers:

```c#
using(Disposable disposable = new Disposable())
{
  disposable.Go();
}
catch(NotSupportedException)
{
  // do something here...
}
```

This would expand to this:

```c#
public void Use()
{
  Disposable disposable = new Disposable();

  try
  {
    disposable.Go();
  }
  catch(NotSupportedException)
  {
    // do something here...
  }
  finally
  {
    if (disposable != null)
    {
      disposable.Dispose();
    }
  }
}
```

I've run into a number of cases where the creation of the object or one of its method calls will throw an exception that I can either handle and/or want to log. But since using doesn't have any concept of exception handlers, I have to revert to writing the handler code myself.

The other feature request is a bit of an oddity. Let's say `Disposable` explicity implemented an interface called `IThingee`:

```c#
public interface IThingee
{
  void Whack();
}

public sealed class Disposable : IDisposable, IThingee
{
  public void Go()
  {
  }
    
  public void Dispose()
  {
  }

  void IThingee.Whack()
  {
  }
}
```

Now I'm stuck. I can't write this code:

```c#
using(IThingee thingee = new Disposable())
{
  thingee.Whack();
}
```

Because the compiler will complain to me: 

```
Error 1 'UsingComplaints.IThingee': type used in a using statement must be implicitly convertible to 'System.IDisposable'
```

But I can't write this either:

```c#
using(Disposable thingee = new Disposable())
{
  thingee.Whack();
}
```

Since `IThingee` is explicity implemented, the compiler barks at me with this:

```
Error 1 'UsingComplaints.Disposable' does not contain a definition for 'Whack'
```

So I'm back to manually writing the `try` ... `finally` handler myself. What would be nice is if I use using on an object that does not "appear" to implement `IDisposable`, add a type check to the code that's generated in the `finally` handler:

```c#
finally
{
  if (thingee != null && thingee is IDisposable)
  {
    (thingee as IDisposable).Dispose();
  }
}
```

This may not be as "performant" ... but I'd have to do this anyway. If C# is willing to generate some code for me, might as well let it generate some more, eh?

Don't get me wrong - I like the `using` statement. Most of the time it does the job. But I've always felt that `using` was added in a somewhat rushed fashion during the pre-1.0 .NET days. The analysis behind the idea is sound, but real-world situations have popped up that I think require `using` to be extended a bit.

> Published: 05.30.2007 06:38:14 AM CST