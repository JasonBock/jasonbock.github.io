---
title: ScopedEventHandler
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# ScopedEventHandler

One idea I've had lately for [Spackle.NET](https://www.nuget.org/packages/Spackle) is to add a class called `ScopedEventHandler`. The motivation for this class comes from this code:

```c#
var provider = new EventProvider();

var handler = new EventHandler((sender, e) => 
  Console.Out.WriteLine("Caught broadcast via manual."));
provider.Broadcast += handler;

try
{
  provider.Speak();
}
finally
{
  provider.Broadcast -= handler;
}
```

If you don't do the "-=" thing, [you can leak memory](https://stackoverflow.com/questions/15593/practical-use-of-system-weakreference). You could use `WeakReference`, but let's stick with the traditional approach but have a disposable object do that for us:

```c#
var provider = new EventProvider();

var handler = new EventHandler((sender, e) => 
  Console.Out.WriteLine("Caught broadcast via scoped."));

using(var scope = new ScopedEventHandler(provider, "Broadcast", handler))
{
  provider.Speak();            
}
```

Now, you may be wondering why I'm passing in a string here, rather than the actual event. Let's take a look at `EventProvider`:

```c#
public class EventProvider
{
  public event EventHandler Broadcast;
    
  public void Speak()
  {
    if(this.Broadcast != null)
    {
      this.Broadcast(this, new EventArgs());
    }
  }
}
```

What I wanted to do was this:

```c#
using(var scope = new ScopedEventHandler(provider.Broadcast, handler))
```

Where the implementation of the constructor looked like this:

```c#
public ScopedEventHandler(EventHandler provider, EventHandler listener)
{
  provider += listener;
}
```

But that lead to this compilation error:

```
Error 1 The event 'Playground.EventProvider.Broadcast' can only appear on the left hand side of += or -= (except when used from within the type 'Playground.EventProvider').
```

This is compiler error [CS0070](https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0070).

So I'm stuck with Reflection:

```c#
public ScopedEventHandler(object provider, string eventName, Delegate listener)
{
  var eventInfo = provider.GetType().GetEvent(eventName);
  eventInfo.AddEventHandler(provider, listener);
}
```

The end result is the client code is clearer and easier in my mind, but the API seems kind of dorky. However, I don't see another way around this. Any suggestions?

> Published: 12.02.2008 11:23:50 AM CST