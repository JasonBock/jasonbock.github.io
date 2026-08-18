---
title: Easy Code Execution Timing
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Easy Code Execution Timing

Last night I had a small revelation when I was trying to get some simple perf numbers for a coding idea I've been running with (which I'll talk about in another post shortly). The "classic" way to do this usually looks like this:

```c#
[TestMethod]
public void MeasureTheOldWay()
{
  Stopwatch watch = Stopwatch.StartNew();
    
  for(int i = 0; i < Constants.Iterations; i++)
  {
    // do interesting code here...
  }
    
  watch.Stop();
  
  this.TestContext.WriteLine("Total time: " + watch.Elapsed);
}
```

But by adding a simple extension method:

```c#
public static class ActionExtensions
{
  public static TimeSpan Time(this Action @this)
  {
    Stopwatch watch = Stopwatch.StartNew();
    @this();
    watch.Stop();
    return watch.Elapsed;
  }
}
```

That transmorgrifies into this:

```c#
[TestMethod]
public void MeasureTheFunctionalWay()
{
  this.TestContext.WriteLine("Total time: " + new Action(() =>
  {
    for(int i = 0; i < Constants.Iterations; i++)
    {
      // do interesting code here...
    }
  }).Time());
}
```

It's a very subtle change, but I really like the second approach, because you don't see how the code is measured. The extension method could completely change what it does to get the total elapsed time and the client doesn't care; it just wants to run some code and get the elapsed time. To be honest, what I did isn't all that earth-shattering - in fact, it's been done before (and for all I know there's other approaches out there on the web). But it's still cool to see how a functional approach changes everything around.

> Published: 07.11.2008 07:47:22 AM CST