---
title: Unions in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unions in .NET

A lot of my .NET research time has been learning ilasm and CIL. It's given me a better perspective on .NET-related issues, and there's one idea that I'd like to discuss: unions. Now, a union is a structure where you can store more than one field into the same storage area. For example, here's a very simple union in C:

```c
union mixed
{
  char c;
  float f;
  int i;
}
```

Note: bonus points goes to the first person who can correctly identify the book that I pulled this example from.

Now, there is no `union`-related keyword in both C# and VB.NET. There is a way you can declare them, though, but first I'm going to show you what unions look like in CIL:

```
assembly extern mscorlib 
{ 
  .publickeytoken = (B7 7A 5C 56 19 34 E0 89) 
  .ver 1:0:37500:0 
} 

.assembly UnionTest 
{ 
  .hash algorithm 0x00008004 
  .ver 1:0:0:0
} 

.module UnionTest.dll 

.namespace UnionTest 
{ 
  .class public sealed explicit SampleUnion 
    extends [mscorlib]System.ValueType 
  { 
    .field [0] public float32 FloatValue 
    .field [0] public int32 IntValue 
    .field [0] public unsigned int8 ByteValue 
  } 
    
  .class public sealed explicit SampleObjectUnion 
    extends [mscorlib]System.ValueType 
  { 
    .field [0] public string StringValue 
    .field [0] public object ObjectValue 
  } 
} 
```

While you may have never looked at ilasm before, you should be able to read this and get a good feel for what's going on. I've declared two value types, `SampleUnion` and `SampleObjectUnion`:

* They extend `System.ValueType` so they can't be used as a base type
* The type is declared with the explicit attribute, which is used to obtain precise layout control of the type's fields
* Each field is declared using the same field slot

If you would compile this CIL via ilasm to produce an assembly, you could use the unions in any .NET language. Here's a code snippet of a C# application using both types:

```c#
using UnionTest

SampleValueUnion svu; 
svu.ByteValue = 44; 
svu.IntValue = 222222; 
svu.FloatValue = 222222.222F; 

SampleReferenceUnion sru; 
sru.StringValue = "Here's a string value."; 
sru.ObjectValue = new System.Collections.Queue(); 
```

Now, you may have wondered why I had two different unions in `UnionTest`. Take a good look at the types within the unions. The types within `SampleValueUnion` are all value types, and `SampleReferenceUnion` contains all reference types. Let's say I tried to do this:

```
.class public sealed explicit SampleUnion 
  extends [mscorlib]System.ValueType 
{ 
  .field [0] public float32 FloatValue 
  .field [0] public int32 IntValue 
  .field [0] public unsigned int8 ByteValue 
  .field [0] public string StringValue 
  .field [0] public object ObjectValue 
}
```

It would compile, but I'd get a `TypeLoadException` when I would try to use it. You can't have both value and reference types in the same field array slot in a union.

As I mentioned before, unions can be declared in C# and VB.NET by using the `StructLayoutAttribute` and `FieldOffsetAttribute` attributes on a value type. Here's what `SampleValueType` looks like in VB.NET:

```vb
Imports System.Runtime.InteropServices
< StructLayout( LayoutKind.Explicit )> _ 
Public Structure SampleUnion 
  < FieldOffset( 0 )> Public FloatValue As Single 
  < FieldOffset( 0 )> Public IntValue As Integer 
  < FieldOffset( 0 )> Public ByteValue As Byte 
End Structure
```

Of course, if both C# and VB.NET had a union keyword, this declaration would be easier to make and read:

```vb
Public Union SampleUnion 
  Public FloatValue As Single 
  Public IntValue As Integer 
  Public ByteValue As Byte 
End Structure
```

Given the fact that you can choose what fields go into which field slots, the syntax may need some tweaking:

```vb
Public Union SampleUnion 
  Public (0) FloatValue As Single 
  Public (0) IntValue As Integer 
  Public (1) ByteValue As Byte 
  Public (1) DoubleValue As Double
End Structure
```

However, I'll leave syntax issues up to the designers of C# and VB.NET if they add unions in a future version. Also, unions are not CLS-compliant, so be careful with your designs if you decide to use them.

By the way, this is a great example showing the power of attributes in .NET. Even though the languages do not explicitly support the declaration of unions with a keyword, you can still create them with metadata.

I hope this article has given you some insight into an aspect of .NET that you may not have been aware of. I encourage you to spend some time learning the base language of .NET: CIL. It'll enhance your understanding of how .NET really works.

> Published: 03.18.2002 12:00:00 AM CST