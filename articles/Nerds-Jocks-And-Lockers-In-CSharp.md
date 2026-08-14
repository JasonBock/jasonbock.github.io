---
title: Nerds, Jocks and Lockers in C#
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Nerds, Jocks and Lockers in C#

I just saw [this post](https://thedailywtf.com/articles/Nerds%2c-Jocks%2c-and-Lockers) on a classic programming problem. I decided to tackle it and here is my solution:

```c#
using System;
using System.Collections.Generic;

namespace NerdsJocksAndLockers
{
  class Program
  {
    static void Main(string[] args)
    {
      var openedLockers = Program.Toggle(100);
      Console.Out.WriteLine(string.Join(", ", openedLockers));
    }
        
    private static string[] Toggle(int lockerCount)
    {
      var lockers = new bool[lockerCount];
      var startingPosition = 0;
            
      for(var i = 1; i <= lockerCount; i++)
      {
        for(var j = startingPosition; j < lockerCount; j = j + i)
        {
          lockers[j] = !lockers[j];
        }
                
        startingPosition++;
      }

      var openedLockers = new List<string>();
            
      for(var x = 0; x < lockerCount; x++)
      {
        if(lockers[x])
        {
          openedLockers.Add(x.ToString());
        }
      }
            
      return openedLockers.ToArray();
    }
  }
}
```

I'm sure there are a **lot** more elegant ways of doing this; this is what I came up with off the cuff. I'm wondering if there's some pattern that can be exploited with the number of lockers so you don't have to manually iterate through all the of the lockers.

Turns out, there is (and it also avoids creating that locker array as well):

```c#
private static string[] ToggleWithNoIterations(int lockerCount)
{
  var openedLockers = new List<string>();
    
  var square = 1;
  var squareResult = (square * square);
  while(squareResult <= lockerCount)
  {
    openedLockers.Add(squareResult.ToString());
    square++;
    squareResult = (square * square);
  }
    
  return openedLockers.ToArray();        
}
```

So which one is faster? Let's time it:

```c#
var random = new SecureRandom();
const int Iterations = 100000;
const int LockerLimit = 1000;

Console.Out.WriteLine("Total Toggle Time: " + new Action(() =>
{
  for(var x = 0; x < Iterations; x++)
  {
    Program.Toggle(random.Next(LockerLimit));
  }
}).Time().ToString());

Console.Out.WriteLine("Total ToggleWithNoIterations Time: " + new Action(() =>
{
    for(var x = 0; x < Iterations; x++)
    {
        Program.ToggleWithNoIterations(random.Next(LockerLimit));
    }
}).Time().ToString());
```

> Note: The `SecureRandom` class and the `Time()` extension method come from my [Spackle](https://www.nuget.org/packages/Spackle) library.

The results?

```
Total Toggle Time: 00:00:01.9582293
Total ToggleWithNoIterations Time: 00:00:00.9138840
```

Again, this is all purely off the cuff; don't trust my results (I'm literally writing this blog post as I experiement!).

> Published: 08.05.2009 11:59:16 AM CST