---
title: Be Careful With Public Abstract Classes ...
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Be Careful With Public Abstract Classes ...

A guy at the Magenic office today ran into a very weird problem with a class within the SharePoint assembly [1]. The class's name is `SPBaseCollection`, but I'm going to illustrate the problem with a simple example. Let's say you have the following C# code in an assembly called `BaseCode` (note that it really doesn't matter if the code was written in C#, but I'm an equal opportunity language user ;) ).

```c#
namespace BaseCode
{
  public abstract class BaseClass
  {
    internal abstract void ImplementThisSucker();

    public void DoSomething() {}
  }

  internal class ImplementingClass : BaseClass
  {
    internal override void ImplementThisSucker()
    {

    }
  }

  public class SomeOtherClass
  {
    public BaseClass GiveOutImplementation()
    {
      return new ImplementingClass();
    }
  }
}
```

Now, you reference this assembly in a VB .NET project (that's an important part), and you think, "aha! I can implement my own version of `BaseClass`!":

```vb
Option Strict On
Option Explicit On 

Imports BaseCode

Public Class ExtendingClass
  Inherits BaseClass

  Private Overrides Sub ImplementThisSucker()

  End Sub
End Class
```

Well ... that doesn't work. Since `ImplementThisSucker()` is `internal`, you can't override it. However, `BaseCode` has to be `public` because `SomeOtherClass` is exposing it through the return value of `ImplementingClass()`. Unfortunately, the error message that the IDE gives when you try to inherit from `BaseClass` is ... well, wrong: "Private Overrides Sub ImplementThisSucker()' cannot override 'Private Overridable MustOverride Sub ImplementThisSucker()' because it is not accessible in this context." `ImplementThisSucker()` in `BaseClass` is not `private` - it's `internal` (or `Friend`)!

The bottom line is that there is no bug here. It feels a little awkward that a public abstract type exists in an assembly that really cannot be extended by any other assembly, but it's technically not incorrect. Also, it's just weird how the IDE displays the error message. Finally, I hope this shows that just because something looks extensible doesn't mean that you can do it.

[1] Take note, recruiters - I am not a SharePoint expert. Nor am I looking for a job, either; it's just weird how you can mention a technology in passing on your blog and suddenly somebody is calling you for your "expertise."

> Published: 06.18.2004 08:15:10 PM CST