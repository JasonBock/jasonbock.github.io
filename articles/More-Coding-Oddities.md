---
title: More Coding Oddities
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# More Coding Oddities

Found this one this morning:

```c#
private string GetActionFromCollection(NameValueCollection values)
{
  string action = values["action"];
  return action;
}
```

This one may look odd at first, but there's an intent behind it: debugging. Take a look at this code:

```c#
using System;

namespace VariableAssignment
{
  class Program
  {
    static void Main(string[] args)
    {
      Console.Out.WriteLine(Program.GetValue());
      Console.Out.WriteLine(Program.GetValueDirectly());
    }
        
    private static int GetValue()
    {
      int value = (new Random()).Next();
      return value;
    }

    private static int GetValueDirectly()
    {
      return (new Random()).Next();
    }
  }
}
```

If you're in the debugger, you can see the value in `GetValue()` before the function returns. In `GetValueDirectly()`, you can't. Now, if I was in VB, this wouldn't be a problem:

```vb
Module Program
  Sub Main()
    Console.Out.WriteLine(Program.GetValue())
    Console.Out.WriteLine(Program.GetValueDirectly())
  End Sub

  Function GetValue() As Integer
    Dim value As Integer = (New Random()).Next()
    Return value
  End Function

  Function GetValueDirectly() As Integer
    Return (New Random()).Next()
  End Function
End Module
```

With VB, I can set a breakpoint in `GetValueDirectly()`, and I'll see the local value change before the function returns.

Now, in both languages, the IL is (essentially) the same (for debug builds) for `GetValueDirectly()`. Here's the IL for the C# version:

```
.method private hidebysig static int32 GetValueDirectly() cil managed
{
  .maxstack 1
  .locals init (
    [0] int32 CS$1$0000)
  L_0000: nop 
  L_0001: newobj instance void [mscorlib]System.Random::.ctor()
  L_0006: callvirt instance int32 [mscorlib]System.Random::Next()
  L_000b: stloc.0 
  L_000c: br.s L_000e
  L_000e: ldloc.0 
  L_000f: ret 
}
```

And here it is for VB (changes in bold):

```
.method public static int32 GetValueDirectly() cil managed
{
  .maxstack 1
  .locals init (
    [0] int32 GetValueDirectly)
  L_0000: nop 
  L_0001: newobj instance void [mscorlib]System.Random::.ctor()
  L_0006: callvirt instance int32 [mscorlib]System.Random::Next()
  L_000b: stloc.0 
  L_000c: br.s L_000e
  L_000e: ldloc.0 
  L_000f: ret 
}
```

Both use a local variable to return the value - the only difference is how it's named between the languages. So there's no reason that C# couldn't show me that local value before it returns ... right? Or am I missing something in the VS debugger?

> Published: 08.29.2007 09:55:06 AM CST