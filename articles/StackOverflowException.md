---
title: `StackOverflowException`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `StackOverflowException`

Yesterday a guy on our team discovered an interesting change to .NET 2.0. He was running some unit tests and (inadvertenly) wrote code that caused a `StackOverflowException`. This actually blew MSTest out of the water, which surprised him. This is a change in .NET 2.0:

> Starting with the .NET Framework version 2.0, a StackOverflowException object cannot be caught by a try-catch block and the corresponding process is terminated by default. Consequently, users are advised to write their code to detect and prevent a stack overflow. For example, if your application depends on recursion use a counter or a state condition to terminate the recursive loop.

Just to check this, I wrote up a small test app:

```c#
using System;

namespace StackOverflowShutdown
{
  class Program
  {
    static void Main(string[] args)
    {
      Console.WriteLine("Invoked.");

      try
      {
        Program.CallMe();            
      }
      catch(StackOverflowException)
      {
        Console.WriteLine("Overflow!");            
      }

      Console.WriteLine("Finished.");
    }
        
    static void CallMe()
    {
      Program.CallMe();
    }
  }
}
```

When I run it, I see, "Invoked.", but then I get the big "this-program-did-something-really-awful" window. I chose not to attach to a debugger, and then the console spit this out:

```
Process is terminated due to StackOverflowException.
```

So I never even got into the exception handler that was set up specifically to handle `StackOverflowException`.

What's kind of odd is that if I turn on CodeAnalysis in this project, it doesn't tell me to consider getting rid of that `catch` block, since it's useless anyway. Hopefully the CodeAnalysis team will (or maybe they already have?) added a rule to let the developer know when catching for certain exceptions is pointless, since the program will terminate anyway.

> Published: 03.29.2007 01:29:45 PM CST