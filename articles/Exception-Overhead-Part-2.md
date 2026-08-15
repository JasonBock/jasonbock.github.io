---
title: Exception Overhead - Part 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Exception Overhead - Part 2

[Yesterday](https://jasonbock.net/articles/Exception-Overhead) I looked at the overhead associated with throwing an exception. Driving into work today, I wondering, "what's the cost involved in creating the exception object?"

So I added this method:

```c#
public sealed class Methods
{
  public void DoGood()
  {
  }
    
  public void DoBad()
  {
    throw new NotImplementedException();
  }

  public void DoSomething()
  {
    var exception = new NotImplementedException();
  }
}
```

And added it to the timings:

```c#
private static void TimeExceptions()
{
  const int iterations = 500000;

  var methods = new Methods();
    
  Console.Out.WriteLine("Bad Action: " + new Action(() =>
  {
    for(var i = 0; i < iterations; i++)
    {
      try
      {
        methods.DoBad();
      }
      catch(NotImplementedException)
      {
      }
    }
  }).Time());

  Console.Out.WriteLine("Good Action: " + new Action(() =>
  {
    for(var i = 0; i < iterations; i++)
    {
      methods.DoGood();
    }
  }).Time());

  Console.Out.WriteLine("Some Action: " + new Action(() =>
  {
    for(var i = 0; i < iterations; i++)
    {
      methods.DoSomething();
    }
  }).Time());
}
```

Here are the results:

```
Bad Action: 00:00:21.3727088
Good Action: 00:00:00.0005447
Some Action: 00:00:04.9421599
```

Creating the object itself takes time. But throwing the exception adds a lot on top of that.

Somebody was wondering if I compared the time between two methods where one calls a stored proc that does nothing, and the other calls a stored proc that throws an exception. I may look into that.

> Published: 04.02.2009 09:26:08 AM CST