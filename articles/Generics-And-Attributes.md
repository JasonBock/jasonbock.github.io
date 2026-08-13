---
title: Generics and Attributes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Generics and Attributes

I really like attributes in .NET. So much so that I co-authored a book on them a long time ago. The only thing that's been kind of annoying is that custom attributes don't support generics ... at least I thought they didn't until today. I ran across [this article](https://stackoverflow.com/questions/294216/why-does-c-sharp-forbid-generic-attribute-types) and I found out you can use generics with attributes, but you can only do it at the IL level. Now, there's a number of things I know you can do in IL that aren't available in languages like C# and VB, and even if you try to define stuff in IL you can't consume it in C# (like overloading methods by return type only), but thankfully generic attributes are consumable in "normal" .NET languages! Here's a simple example on how you can do this.

Let's say you want to have a generic attribute like this:

```c#
[AttributeUsage(AttributeTargets.Class)]
public abstract class MyGenericAttribute<T> : Attribute
{
  protected MyGenericAttribute();

  public abstract void Consume(T target);
}
```

Rather than trying to figure out the IL by hand, I created two classes in C#, one that defines the attibute, and one that defines the generic portion:

```c#
[AttributeUsage(AttributeTargets.Class)]
public abstract class MyGenericAttribute : Attribute { }

public abstract class MyGenericAttributeMock<T>
{
  public abstract void Consume(T target);
}
```

Then I compiled this code, cracked it open in ILDasm, and dumped the contents to a .il file. Then I tweaked the IL to get one class definition:

```
.assembly extern mscorlib
{
  .publickeytoken = (B7 7A 5C 56 19 34 E0 89 )
  .ver 4:0:0:0
}

.assembly GenericAttributesFromIL
{
  .hash algorithm 0x00008004
  .ver 1:0:0:0
}

.module GenericAttributesFromIL.dll
.imagebase 0x00400000
.file alignment 0x00000200
.stackreserve 0x00100000
.subsystem 0x0003
.corflags 0x00000001

.class public abstract auto ansi 
  beforefieldinit GenericAttributesFromIL.MyGenericAttribute`1<T>
  extends [mscorlib]System.Attribute
{
  .custom instance void [mscorlib]System.AttributeUsageAttribute::.ctor(
    valuetype [mscorlib]System.AttributeTargets) = ( 01 00 04 00 00 00 00 00 ) 

  .method public hidebysig newslot abstract virtual 
    instance void Consume(!T target) cil managed { }

  .method family hidebysig specialname rtspecialname 
    instance void .ctor() cil managed
  {
    .maxstack  8
    IL_0000:  ldarg.0
    IL_0001:  call instance void [mscorlib]System.Attribute::.ctor()
    IL_0006:  ret
  }
}
```

As you can see, `MyGenericAttribute` is now generic. I compiled this code, and referenced the assembly in a console app. Now I can define a custom attribute like this:

```c#
[AttributeUsage(AttributeTargets.Class)]
public sealed class TargetAttribute : MyGenericAttribute<string>
{
  public override void Consume(string target)
  {
    Console.Out.WriteLine("I consumed " + target);
  }
}
```

Slick! I can add this attribute to a class like this:

```
[Target]
public sealed class SomeTarget { }
```

And it works just fine.

If you really need to have generic attributes, try doing the IL dance I demonstrated above. You may be able to add a nice dimension to your designs.

> Published: 07.11.2011 02:50:59 PM CST