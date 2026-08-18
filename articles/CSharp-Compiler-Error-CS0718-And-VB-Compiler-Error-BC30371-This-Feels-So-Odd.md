---
title: C# Compiler Error CS0718 and VB Compiler Error BC30371 - This Feels So ... Odd
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# C# Compiler Error CS0718 and VB Compiler Error BC30371 - This Feels So ... Odd

> UPDATE (08.01.2008 - 10:36 AM) - After reading Eric's comments...he's right, you can instantiate `SealedClass` (think singleton pattern :) ), so I changed the wording of the post around a bit. But I still think the behavior is odd, especially since you can use `ModuleCode` in C#.

Consider the following two classes:

```c#
public static class StaticClass
{
}

public sealed class SealedClass
{
  private SealedClass() : base()
  {
  }
}
```

Now, I thought there wasn't any difference between these definitions. There are some slight deltas, though ... here's the dump from ILDasm for both classes:

```
.class public abstract auto ansi sealed beforefieldinit CSharpCode.StaticClass
  extends [mscorlib]System.Object
{
}

.class public auto ansi sealed beforefieldinit CSharpCode.SealedClass
  extends [mscorlib]System.Object
{
  .method private hidebysig specialname rtspecialname 
    instance void  .ctor() cil managed
  {
    .maxstack  8
    IL_0000:  ldarg.0
    IL_0001:  call       instance void [mscorlib]System.Object::.ctor()
    IL_0006:  ret
  }
}
```

It's odd that the static class is defined both as abstract and sealed, but ... oh well. Anyway, let's define a generic class:

```c#
public sealed class GenericClass<T>
{
}
```

OK, here's the oddity. The following call works:

```c#
var genericSealed = new GenericClass<SealedClass>();
```

But this one doesn't:

```c#
var genericStatic = new GenericClass<StaticClass>();
```

I get the following compilation error:

```
Error 1 'CSharpCode.StaticClass': static types cannot be used as type arguments C:\JasonBock\Personal\.NET Projects\CS0718\CSharpCode\GenericClassConsumer.cs 12 41 CSharpCode
```

Here's what the documentation says for [CS0718](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/generic-type-parameters-errors?redirectedfrom=MSDN#generic-type-usage-restrictions):

> Because a static type cannot be instantiated, it cannot be used as a generic argument. To resolve this error, remove the static type from the generic argument.

But...if you create an abstract class:

```c#
public abstract class AbstractClass
{
}
```

And use that in the generic type:

```c#
var genericAbstract = new GenericClass<AbstractClass>();
```

That works.

I can even create a "module" type in VB:

```vb
Public Module ModuleCode

End Module
```

Which looks like this in IL:

```
.class public auto ansi sealed VBCode.ModuleCode
  extends [mscorlib]System.Object
{
  .custom instance void
    [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.StandardModuleAttribute::.ctor() = 
      ( 01 00 00 00 ) 
} // end of class VBCode.ModuleCode
```

It's pretty much like `StaticClass`, but without the `abstract` qualifier. And guess what, I can use that with the generic type:

```c#
var genericModule = new GenericClass<ModuleCode>();
```

As a side note, if I reverse the issue and write this in VB:

```vb
Public Module VBGenericClassConsumer
  Public Sub Consume()
    Dim genericSealed As New GenericClass(Of SealedClass)()
    Dim genericAbstract As New GenericClass(Of AbstractClass)()
    Dim genericStatic As New GenericClass(Of StaticClass)()
    Dim genericModule As New GenericClass(Of ModuleCode)()
  End Sub
End Module
```

That last line of code is invalid - I get the following error message:

```
Error 1 Module 'ModuleCode' cannot be used as a type. C:\JasonBock\Personal\.NET Projects\CS0718\VBCodeThatReferencesEverything\Stuff.vb 9 45 VBCodeThatReferencesEverything
```

This is compiler error [BC30371](https://learn.microsoft.com/en-us/dotnet/visual-basic/misc/bc30371).

So ... these compiler errors are really odd. I'm guessing in the C# it must be looking for types that are abstract and sealed with no constructors and considering those to be "static types". With VB, I'm guessing that it's looking for classes that have the `StandardModuleAttribute`. Why? Because I compiled the following IL code:

```
.assembly extern mscorlib
{
  .publickeytoken = (B7 7A 5C 56 19 34 E0 89 )
  .ver 2:0:0:0
}

.assembly ILStuff
{
  .custom instance void 
    [mscorlib]System.Runtime.CompilerServices.CompilationRelaxationsAttribute::.ctor(int32) = 
        ( 01 00 08 00 00 00 00 00 ) 
  .custom instance void 
    [mscorlib]System.Runtime.CompilerServices.RuntimeCompatibilityAttribute::.ctor() = 
        ( 01 00 01 00 54 02 16 57 72 61 70 4E 6F 6E 45 78   
        63 65 70 74 69 6F 6E 54 68 72 6F 77 73 01 )       
  .hash algorithm 0x00008004
  .ver 1:0:0:0
}

.module ILStuff.dll
.imagebase 0x00400000
.file alignment 0x00000200
.stackreserve 0x00100000
.subsystem 0x0003       
.corflags 0x00000001    

.class public auto ansi sealed NonAttributedModuleCode
  extends [mscorlib]System.Object
{
}
```

And I was able to use `NonAttributedModuleCode` as a generic argument in both C#:

```c#
var genericNonAbstractSealed = new GenericClass<NonAttributedModuleCode>();
```

And VB:

```vb
Dim genericNonAttributedModule As New GenericClass(Of NonAttributedModuleCode)()
```

At the end of the day, the compilers are "working by design". But ... the design doesn't feel right in either language. With VB, it seems like it's looking for a specific attribute in the class (but without that attribute the code works so it seems like an odd restriction), but in C#, there are other types that match the description in the compiler ("cannot be instantiated") that work just fine.

Thoughts? Opinions? Criticism?

> Published: 07.23.2008 09:26:39 AM CST