---
title: An Interesting Discussion About `CInt()`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# An Interesting Discussion About `CInt()`

Today on the internal technical lists at Magenic, a conversation started on the merits between C# and VB .NET. To be honest, these discussions go nowhere and end up with everyone pretty much saying, "pick the language you like, unless the client insists on something else." However, this conversation started to get into the VB .NET helper classes and what really goes on when you pick one of the conversion functions in VB .NET.

To start, let's say you want to convert a string into an integer. There's a lot of ways to tackle this problem, but let's stick to the handy-dandy `CInt()` function:

```vb
Private Sub StringCInt()
  Dim s As String = "3"
  Dim x As Integer = CInt(s)
End Sub
```

If you translate this to CIL, you get this:

```
.method private instance void StringCInt() cil managed
{
// Code Size: 14 byte(s)
.maxstack 1
.locals (string V_0, int32 V_1)
L_0000: ldstr "3"
L_0005: stloc.0 
L_0006: ldloc.0 
L_0007: call int32 [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.IntegerType::FromString(string)
L_000c: stloc.1 
L_000d: ret 
}
```

(Note: All of the CIL was taken from Reflector - you owe it to yourself to get the latest version). As you can see, `CInt()` gets translated into a call to `IntegerType.FromString()`. However, the VB .NET compiler doesn't always use the classes in the `Microsoft.VisualBasic.CompilerServices` namespace when a conversion occurs. Take a look at this VB .NET code:

```vb
Private Sub DoubleCInt()
  Dim d As Double = 3.2
  Dim y As Integer = CInt(d)
End Sub
```

The generated opcodes for `DoubleCInt()` are as follows:

```
.method private instance void DoubleCInt() cil managed
{
// Code Size: 19 byte(s)
.maxstack 1
.locals (float64 V_0, int32 V_1)
L_0000: ldc.r8 3.2
L_0009: stloc.0 
L_000a: ldloc.0 
L_000b: call float64 [mscorlib]System.Math::Round(float64)
L_0010: conv.ovf.i4 
L_0011: stloc.1 
L_0012: ret 
}
```

In this case, the compiler doesn't use any of VB .NET's compiler helper classes; it converts the double into an integer via `Math.Round()`.

Finally, if you're converting a long to an integer:

```vb
Private Sub LongCInt()
  Dim l As Long = 3
  Dim q As Integer = CInt(l)
End Sub
```

the resulting CIL is quite efficient - it doesn't use any function calls at all!

```
.method private instance void LongCInt() cil managed
{
// Code Size: 14 byte(s)
.maxstack 1
.locals (int64 V_0, int32 V_1)
L_0000: ldc.i8 3
L_0009: stloc.0 
L_000a: ldloc.0 
L_000b: conv.ovf.i4 
L_000c: stloc.1 
L_000d: ret 
}
```

So what's the point of this? Well, the discussion centered around using `Convert.ToInt32()` and `Int32.Parse()` over `CInt()`. To make a long story short, `Convert.ToInt32()` simply forwards the data to `Int32.Parse()` when you're converting string data to an integer. Some quick performance checking code shows that using either BCL method is 2 to 4 times faster than using `CInt()`. However, if you're trying to convert a long to an integer, `CInt()` wins over the BCL method approaches (`CInt()` is about 15% to 25% faster than `Convert.ToInt32()`). Therefore, in my mind it's a trade-off. You can use VB-centric approaches to your code, but it definitely helps to know what's going on underneath the hood. Granted, I would run performance tests on the code to see where the bottlenecks are first before I analyze CIL, but in this case I was more interested in what the compilers were doing with VB .NET's conversion functions [1]. By going through this little investigation, I now know when using `CInt()` is a good choice, and when the BCL approach is the winner.

One interesting side note about all of this. Obviously in C# there are no conversion functions like `CInt()`, so you have no choice but to use the BCL approach. However, in the case of converting a long to an integer, one can argue that a simple cast is the way to go:

```c#
public void LongCastToInt()
{
  long l = 3;
  int z = (int)l;
}
```

The C# compiler creates these opcodes:

```
.method public hidebysig instance void LongCastToInt() cil managed
{
// Code Size: 4 byte(s)
.maxstack 1
.locals (int64 V_0)
L_0000: ldc.i4.3 
L_0001: conv.i8 
L_0002: stloc.0 
L_0003: ret 
}
```

The key difference to note between the CIL that the C# and VB .NET compilers generate is that the C# CIL results doesn't store the value of 3 into a local variable and reload it, whereas the VB .NET CIL does, even in release mode with optimizations turned on. Granted, the real key is what the x86 does, and in my testing the C# cast approach is slightly faster than the VB .NET `CInt()` approach (although it's really slim - about 0.01% faster) [2].

[1] I haven't looked at every VB .NET conversion function, so I have no idea what the compiler does with the other Cxxx() "methods".

[2] By the way, there's no difference in the CIL when you're converting a long to an int using either `CInt()` or a cast via `CType()`.

> Published: 06.03.2004 12:18:53 AM CST