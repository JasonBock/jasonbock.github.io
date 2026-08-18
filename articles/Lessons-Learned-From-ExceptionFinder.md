---
title: Lessons Learned From ExceptionFinder
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Lessons Learned From ExceptionFinder

Lately I've been updating my [ExceptionFinder](https://github.com/JasonBock/ExceptionFinder) project. It's similar to my [DynamicProxies](https://github.com/JasonBock/DynamicProxies) project in that it requires knowledge of IL. Even though I [wrote a book on IL](https://link.springer.com/book/10.1007/978-1-4302-0845-7), I don't work with it every single day, so I tend to forget a lot of the nuances. Anyway, I've run into a lot of interesting bugs and scenarios trying to get this add-in working correctly, and I've decided to post them. I know that authoring Reflector add-ins is not done by a lot of people, but the things I've run into aren't always IL-related and hopefully the lessons are beneficial.

## Removing Dependencies in Tests That You Have No Control Over Isn't Easy

I initially wrote tests that covered very specific, simple situations. e.g the tests in `ThrowingScenariosTests`. This gave me the exact control I need to ensure I reported the right number of leaked exceptions from code generated from the VB and C# compilers. This may not seem like a big deal, but if you look at my test cases, I also generate an assembly via Reflection.Emit (e.g. `ExceptionLocationLoadingScenariosTests`). As I discovered bugs, I found out that having this precise control was essential in being able to reproduce the situations that brought the bug to light.

However, even though I write my own assemblies, I didn't always do that. Case in point: `AnalyzerTests`. I originally did it to fix bugs that I saw when I tested the add-in in the UI - i.e. I'd see something odd happen with a method in mscorlib, so I'd write a test against that method directly and figure out what the problem was. What I should've done is **always** isolate the problem in code I either emit or write myself. Therefore, once the bug was ironed out, write code that reproduces the bug, write a test around that, and delete the mscorlib-dependent tests. The problem is that I have no control over these methods, and it's possible that future releases will change the underlying implementation. I need to remove them and ensure I still have the same code coverage and logic paths covered. This is still a work in progress, and it may not be easy to extract ...

## Only Test What Is Valid

Sometimes in my tests I'd have assertions on getting an exact number of exceptions. That has two problems: a future version of the C# and/or VB compilers may cause that number to change, and during refactorings I may fix a bug that causes the test to fail...because it's actually reporting more leaked exceptions like it should. Here's an example: VB will insert code into other methods in exceptions handlers. For example, this code:

```vb
Public Function TryWithCatchAndFilterThatMightThrowException( _
  ByVal denominator As Integer) As Integer
 
  Dim x As Integer

  Try
    x = 3 \ denominator
  Catch ex As DivideByZeroException _
    When FilterHandlerScenarios.DoesThisThrowException(2)
 
 Throw New NotSupportedException("Can't divide by zero.", ex)
  End Try

  Return x
End Function
```

turns into this in IL:

```
.method public static int32 
  TryWithCatchAndFilterThatMightThrowException(int32 denominator) cil managed
{
  .maxstack 3
  .locals init (
    [0] int32 TryWithCatchAndFilterThatMightThrowException,
    [1] int32 x,
    [2] class [mscorlib]System.DivideByZeroException ex)
  L_0000: ldc.i4.3 
  L_0001: ldarg.0 
  L_0002: div 
  L_0003: stloc.1 
  L_0004: leave.s L_0031
  L_0006: isinst [mscorlib]System.DivideByZeroException
  L_000b: dup 
  L_000c: brtrue.s L_0012
  L_000e: pop 
  L_000f: ldc.i4.0 
  L_0010: br.s L_0022
  L_0012: dup 
  L_0013: stloc.2 
  L_0014: call void 
    [Microsoft.VisualBasic]Microsoft.VisualBasic.CompilerServices.ProjectData::SetProjectError(
    class [mscorlib]System.Exception)
  L_0019: ldc.i4.2 
  L_001a: call bool 
    ExceptionFinder.Tests.FilterScenarios.FilterHandlerScenarios::DoesThisThrowException(int32)
  L_001f: ldc.i4.0 
  L_0020: cgt.un 
  L_0022: endfilter 
  L_0024: pop 
  L_0025: ldstr "Can\'t divide by zero."
  L_002a: ldloc.2 
  L_002b: newobj instance void 
    [mscorlib]System.NotSupportedException::.ctor(
    string, class [mscorlib]System.Exception)
  L_0030: throw 
  L_0031: ldloc.1 
  L_0032: ret 
  .try L_0000 to L_0006 filter L_0006 handler L_0024 to L_0031
}
```

Notice the call to `SetProjectError()`? When I fixed a bug, it caused the leaked exception count to jump. But what I really cared about is this: I only wanted to see if the exceptions I expected were in the collection. I don't really care about the total count. The point is, make sure a test only tests for the results you want to see.

## Exception Objects Can Come From Anywhere

In short, I originally made some really bad assumptions that exceptions were always created via a `newobj` opcode. Well ... they are, but that doesn't mean the object itself can't get on the stack some other way (i.e. they can be return arguments from methods, they can be arguments to methods, they can be fields ... ). My original implementation was so wrong it was definitely not right.

## Beware of NOPs!

I made a terrible assumption in my analyzer code with respect to the throw opcode. When I reviewed the documentation for it, I read way too much into the following section:

> The stack transitional behavior, in sequential order, is:
> 
> 1. An object reference (to an exception) is pushed onto the stack.
> 2. The object reference is popped from the stack and the exception thrown.

Of course, it doesn't mean that there can't be 37 `nop` opcodes between the exception object getting on the stack and the throw opcode!

The `ldarg` and `ldarg.s` opcodes will end up getting `nop` opcodes thrown in between (see `ThrowExceptionFromLdargInstruction()` and `ThrowExceptionFromLdargSInstruction()`), even though I don't put them in there (I have a test that explicitly injects lots of `nop` opcodes so I have that covered). But that was a good thing, as I made a false assumption that the previous instruction was always the right one - i.e. it put the object to be thrown on the stack.

That's why in `ListOfInstructionsExtensions` there's now a `GetValidPreviousInstruction()` method, which skips `nop` opcodes.

## New Ideas May Uncover Unexpected "Behaviors"

The "let's parse the XML comments" idea to find reported exceptions led me to find a bug. The XML comments for the `Create()` method on `System.IO.File` said it could throw a number of exceptions ... but my analyzer said it didn't throw any of those. That didn't make any sense. My original intent was to show the found exceptions in different ways, and I'm still going to do that, but starting the work to support this feature led me to fix another bug. Which was ...

## Oh Yeah, `newobj` is a Method Call Instruction - Duh!

The reason my analyzer wasn't reporting exceptions for `File.Create()` call was that I didn't consider the `newobj` opcode to be a method call, which it most definitely it (it calls a constructor). Oddly enough, adding this one iddy-biddy piece of code (adding an OR check in `IsMethodCall()`) really stressed out my stress test. What used to take 3 minutes now takes 10 minutes!

## I Figured Out A Use for the .vsmdi File

Speaking of stress tests ... I really wanted the ability to have all kinds of test categories, but it didn't seem like MSTest supports this. But, you can do it via a .vsmdi file. Nice - I can different kinds of test sets, like unit, stress, performance, integration, etc. and run the ones I want when I want. It's actually kind of useful, and it seems to have stopped the 2.vsmdi, 3.vsmdi, 4.vsmdi, etc. nonsense as well.

## Getting the Type Name From `ITypeDeclaration`'s Name Property Doesn't Always Work

A generic type will come back as "TypeName<T>". That's incorrect if you want to pass it to `Type.GetType()`. It should be "TypeName`1", where the number equals the number of generic arguments. This was an easy fix in `ITypeReferenceExtensions` - I just added a `GetName()` extension method.

I'm sure I still have lots of lessons to learn from this project. But I believe that working so deeply with assemblies helps in getting a better perspective on the .NET landscape. That's why I keep doing it.

> Published: 08.27.2008 02:34:29 PM CST