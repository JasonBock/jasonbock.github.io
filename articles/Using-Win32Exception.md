---
title: Using `Win32Exception`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using `Win32Exception`

I've had a couple of readers send me explanations about the description problem from 07.19.2002. Remember, though, the issue I'm having is that description should be set after the constructor is done, and it's not. I'm still following up on some ideas that have sent, though, so stay tuned.

By the way, here's a small piece of C# code illustrating the usage of `Win32Exception`:

```c#
using System.ComponentModel;

class Win32Test
{
  [STAThread]
  static void Main(string[] args)
  {
    try
    {
      throw new Win32Exception(5);
    }
    catch(Exception e)
    {
      Console.WriteLine(e.Message);
    }
  }
}
```

This prints out "Access is denied" to the console. I didn't know this class existed until a couple of days ago (this illustrates just how vast the .NET Framework API is). Because of this, I ended up using P/Invokes to `FormatMessage()` in my ".NET Security" book, which takes more work than the one line of code shown here ;).

> Published: 07.22.2002 11:05:00 AM CST