---
title: Fun With Exception-Based Stack Traces
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Fun With Exception-Based Stack Traces

A while ago, I created an error dialog window that parsed the information from an `Exception` object. The app that I'm working on wanted to include it, so we took the code and tweaked it a bit (and also ported it from C# to VB .NET). One of the tweaks was kind of interesting - it was related to the stack trace UI display code. Here's what it currently looks like from the code on my site (I'll keep the discussion in C#):

```c#
string[] stackTrace =
  targetException.StackTrace.Split(new char[] {'\n'});

foreach(string st in stackTrace)
{
  this.stackTraceList.Items.Add(new ListViewItem(st));
}
```

The problem was that the `Split()` call wasn't good enough, because I needed to remove more than what I was doing. I ended up with an ASCII square in the list view, and it looked kind of awkward. Furthermore, all I get back is a string that represents the method that was called. When I moved this into the in-house app, I wondered if the `StackTrace` had anything to help me parse the exception's stack trace. To my suprise, it does!

```c#
StackTrace exStack =
  new StackTrace(targetException);

for(i = 0; i < exstack.FrameCount; i++)
{
  StackFrame exFrame = exStack.GetFrame(i);
  this.stackTraceList.Items.Add(new ListItemView(exFrame.GetMethod().Name));
}
```

The nice thing about this approach is that you can actually get at a `MethodBase` reference. I don't show it here, but we ended up adding a lot more to the list view about each method in the stack that what could be done with the first approach. Granted, it would be nice if I could do a `foreach` on the `StackTrace` object, but this is still a better approach in my book.

I've been using/working with .NET for years now, and I'm still finding cool things within it.

> Published: 10.14.2003 03:25:09 PM CST