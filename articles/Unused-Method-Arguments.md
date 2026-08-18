---
title: Unused Method Arguments
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unused Method Arguments

Last night at Omaha.NET I got an interesting question during my talk. It was based around code that looked like this:

```c#
public int DoSomething(int x, int y)
{
  return x * x;
}
```

Basically I made the claim that VS will give you an error, saying that y isn't being used. Someone in the audience said that was incorrect - the compiler wouldn't bark at you. I couldn't sworn I saw the IDE yell at me when I've done this before ... but I needed to do it in VS to be sure. So I got home and tried it out.

Turns out we were both right :)

He was correct in that the C# compiler won't complain about it. But if you have CodeAnalysis turned on (with warnings as errors enabled), you'll see CA1808 pop up:

```
CA1801 : Microsoft.Usage : Parameter 'y' of 'Beer.DoSomething(int, int)' is never used. Remove the parameter or use it in the method body.
```

Since I have CodeAnalysis cranked up in my projects, that's why I see it.

> Published: 08.29.2008 03:49:23 PM CST