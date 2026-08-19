---
title: What's the Magic Behind Extension Methods?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# What's the Magic Behind Extension Methods?

An attribute.

Here's what I'm getting at. Before C# 3.0, you could basically do the same thing extension methods do, except the syntax looks a little different:

```c#
internal sealed class Program
{
  private static void Main(string[] args)
  {
    Target t = new Target();    
    TargetExtensions.Implicit(t);
  }
}

public sealed class Target
{
}

public static class TargetExtensions
{
  public static void Implicit(Target target)
  {
  }    
}
```

With the extension method syntax, it now looks like the method is available on the object directly:

```c#
internal sealed class Program
{
  private static void Main(string[] args)
  {
    Target t = new Target();    
    TargetExtensions.Implicit(t);
    t.Explicit();
  }
}

public sealed class Target
{
}

public static class TargetExtensions
{
  public static void Implicit(Target target)
  {
  }    
    
  public static void Explicit(this Target target)
  {
  }
}
```

But in IL, the methods look exactly the same:

```
.method public hidebysig static void Implicit(class ExtensionsPlayground.Target target) cil managed

.method public hidebysig static void Explicit(class ExtensionsPlayground.Target target) cil managed
So what's the difference?

.method public hidebysig static void Implicit(class ExtensionsPlayground.Target target) cil managed
{
  .maxstack 8
  L_0000: nop 
  L_0001: ret 
}

.method public hidebysig static void Explicit(class ExtensionsPlayground.Target target) cil managed
{
  .custom instance void [System.Core]System.Runtime.CompilerServices.ExtensionAttribute::.ctor()
  .maxstack 8
  L_0000: nop 
  L_0001: ret 
}
```

The `ExtensionAttribute` is added to extension methods. That's the magic sauce that C# will look for to make your life a little easier from a typing perspective. 

This is nice, but I'm still waiting for the day when we as developers can do our own compile- and run-time magic with our own custom attributes. Granted, we probably won't be able to change the language around but just having better support for compiler (Phoenix?) and run-time modifications (LinFu, etc.) would open up a lot of doors ...

As a side note, you can do this in VB, but you have to use the attribute explicity:

```vb
Option Strict On
Option Explicit On

Imports System.Runtime.CompilerServices

Public Module VBTargetExtensions
  <Extension()> _
  Public Sub VBExplicit(ByVal target As VBTarget)

  End Sub

  Public Sub Implicit(ByVal target As VBTarget)

  End Sub

  Public Sub DoIt()
    Dim t As New VBTarget()
    t.VBExplicit()
    VBTargetExtensions.Implicit(t)
  End Sub
End Module

Public Class VBTarget

End Class
```

You say "tomato", I say "tomato". It all works the same in the end.

What's nice is that if your language supports attributes but not extension methods directly, you can still write extension methods and have them work in languages that understand what to do with that attribute.

> Published: 01.14.2008 07:58:58 AM CST