---
title: Exception Overhead
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Exception Overhead

One of the things I demonstrate in my "Exceptional Development" talk is the price you pay in performance when you throw an exception. That doesn't mean that exceptions are bad; quite the opposite, exceptions are very useful (when used right!). But, as a developer, you do have to keep in mind that throwing an exception causes the CLR to do some incredibly advanced code called "fancy stuff" to fill up the exception with useful information, get SEH involved, and so on.

Just how much overhead is this? I decided to whip up the simplest example I could possibly think of to time this overhead. I wrote two methods, one that did nothing, and one that threw an exception:

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
}
```

Then I timed how long it would take to invoke these methods:

```c#
class Program
{
  static void Main(string[] args)
  {
    Program.TimeExceptions();
  }

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
  }
}
```

> Note: that `Time()` method is an extension method I wrote for `Action` - it's in the [Spackle](https://github.com/JasonBock/SpackleNet) library.

So, what are the results?

```
Bad Action: 00:00:21.3489859
Good Action: 00:00:00.0017535
```

Wow. That's like, what, 5 orders of magnitude worse!

Now, again, keep in mind that I'm not saying you should never throw an exception. The last thing I'd want to hear in the future is somebody saying "but you said on your blog that exceptions make your application's performance degrade!" The key is this: Exceptions should only be thrown in exceptional circumstances. Therefore, the number of times an exception is thrown should be minimal (i.e. you shouldn't be writing code that uses exception handling for program logic and flow).

> Published: 04.01.2009 01:05:23 PM CST