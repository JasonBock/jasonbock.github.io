---
title: Reaching Unreachable Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reaching Unreachable Code

Take a look at the following C# and VB code snippets:

```c#
public sealed class WhatWillItDo
{
  public bool DoIt()
  {
    return true;
    this.DoSomethingElse();
  }

  private void DoSomethingElse() {}
}
```

```vbnet
Public NotInheritable Class WhatWillItDo
  Public Function DoIt() As Boolean
    Return True
    Me.DoSomethingElse()
  End Function

  Private Sub DoSomethingElse()
  End Sub
End Class
```

They look identical, right? OK, they also look asanine, but the issue isn't that they don't really do anything. The issue is the return keyword in VB and how the compiler handles.

In C#, if you have compiler warnings as errors turned on, the code won't compile - you'll get a "unreachable code detected" message. But, even with the same project settings in VB, you don't.

It gets even weirder if you look at the IL:

```
.method public instance bool DoIt() cil managed
{
  .maxstack 1
  .locals init (
    [0] bool DoIt)
  L_0000: nop 
  L_0001: ldc.i4.1 
  L_0002: stloc.0 
  L_0003: br.s L_000c
  L_0005: ldarg.0 
  L_0006: callvirt instance void VBVersion.WhatWillItDo::DoSomethingElse()
  L_000b: nop 
  L_000c: ldloc.0 
  L_000d: ret 
}
```

Using Reflector, this is what it looks like in C#:

```c#
public bool DoIt()
{
  bool DoIt;
  return true;
  this.DoSomethingElse();
  return DoIt;
}
```

For some reason it doesn't get rid of the code after the return; in fact, it includes it, but then it skips over it.

If you change the VS setting from "Debug" to "Release" it changes again - here's the IL:

```
.method public instance bool DoIt() cil managed
{
  .maxstack 1
  .locals init (
    [0] bool DoIt)
  L_0000: ldc.i4.1 
  L_0001: ret 
}
```

And translated into C#:

```c#
public bool DoIt()
{
  return true;
}
```

To me, this is bizarre. I have no idea why VB acts like this in debug mode. Any ideas?

> Published: 09.28.2010 04:37:58 PM CST