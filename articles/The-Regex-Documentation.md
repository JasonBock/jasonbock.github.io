---
title: The Regex Documentation
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Regex Documentation

Today a co-worker pointed out the following line in the documentation for Regex:

> Using a static method is equivalent to constructing a **Regex** object, using it once and then destroying it.

Um ... what??

That doesn't jive with the following code:

```c#
class TestClass
{
  [STAThread]
  static void Main(string[] args)
  {
    StaticMethodApproach.CallMe();
    (new InstanceMethodApproach()).CallMe();
    StaticMethodApproach.CallMe();
    (new InstanceMethodApproach()).CallMe();        
    StaticMethodApproach.CallMe();
    (new InstanceMethodApproach()).CallMe();
  }
}

class StaticMethodApproach
{
  static StaticMethodApproach()
  {
    Console.WriteLine("StaticMethodApproach::.cctor() was invoked.");
  }

  public static void CallMe()
  {
    Console.WriteLine("StaticMethodApproach::CallMe() was invoked.");
  }
}

class InstanceMethodApproach
{
  public InstanceMethodApproach()
  {
    Console.WriteLine("InstanceMethodApproach::.ctor() was invoked.");
  }

  public void CallMe()
  {
    Console.WriteLine("InstanceMethodApproach::CallMe() was invoked.");
  }
}
```

If you run this, you should see the following output:

```
StaticMethodApproach::.cctor() was invoked.
StaticMethodApproach::CallMe() was invoked.
InstanceMethodApproach::.ctor() was invoked.
InstanceMethodApproach::CallMe() was invoked.
StaticMethodApproach::CallMe() was invoked.
InstanceMethodApproach::.ctor() was invoked.
InstanceMethodApproach::CallMe() was invoked.
StaticMethodApproach::CallMe() was invoked.
InstanceMethodApproach::.ctor() was invoked.
InstanceMethodApproach::CallMe() was invoked.
Press any key to continue
```

The key point to see is that the type initializer for `StaticMethodApproach` is only called once. The constructor for `InstanceMethodApproach` is called every time you invoke `CallMe()`. In other words, the type is only loaded once, but you get three instances of `InstanceMethodApproach`. That doesn't seem "equivalent" to me.

Instead of "equivalent", I would go with "analogous", although even that seems to be too strong of a connection between the concepts. Also, I checked the .NET 2.0 docs and it hasn't changed. I don't know, this just doesn't feel like the right wording to me.

> Published: 12.22.2004 01:56:04 PM CST