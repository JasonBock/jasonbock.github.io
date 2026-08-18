---
title: Throwing Exceptions That Are Not Exceptions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Throwing Exceptions That Are Not Exceptions

As I was reading the chapter on exceptions in "Framework Design Guidelines", I got to section 7.2.2, and here's one of their recommendations:

> DO NOT handle non-CLS compliant exception using a parameterless catch block.

It's not a well-known feature of the CLR, but languages **can** throw exceptions that do not derive from `Exception` (C++ is an example of a language that can do this). Granted, running into a case where you'll get an exception that doesn't derive from `Exception` is extrememly rare - I've never run into a situation where I've seen an exception that isn't an exception. This is probably due to the fact that C# and VB - the two most popular .NET langauges - do not allow a developer to throw something that isn't a "real" exception. However, after I played around with some IL tonight, I'm not sure I completely agree with this recommendation.

Consider the following code:

```c#
using System;

public class BadClass
{
  public void BadMethod()
  {
    throw new Exception();
  }
}
```

What I'd like to do to work with non-`Exception` types is to do this:

```c#
using System;

public class BadClass
{
  public void BadMethod()
  {
    throw new object();
  }
}
```

Of course, that's not legal C#. Therefore, I compiled the first version of `BadClass`, opened it up in ILDasm, dumped the IL, and tweaked the IL like this:

```
.assembly extern mscorlib
{
  .publickeytoken = (B7 7A 5C 56 19 34 E0 89 )                         
  .ver 1:0:5000:0
}
.assembly BadClass
{
  .hash algorithm 0x00008004
  .ver 0:0:0:0
}

.module BadClass.dll
.imagebase 0x00400000
.subsystem 0x00000003
.file alignment 512
.corflags 0x00000001

.class public auto ansi beforefieldinit BadClass
       extends [mscorlib]System.Object
{ } 

.class public auto ansi beforefieldinit BadClass
       extends [mscorlib]System.Object
{
  .method public hidebysig instance void 
          BadMethod() cil managed
  {
    .maxstack  1
    IL_0000:  newobj instance void [mscorlib]System.Object::.ctor()
    IL_0005:  throw
  }

  .method public hidebysig specialname rtspecialname 
          instance void  .ctor() cil managed
  {
    .maxstack  1
    IL_0000:  ldarg.0
    IL_0001:  call  instance void [mscorlib]System.Object::.ctor()
    IL_0006:  ret
  }
}
```

Focus on the code in `BadMethod()`. Note that `BadMethod()` is throwing the object that was loaded on the stack, rather than the `System.Exception` that was being thrown. Finally, I compiled the code via ilasm.

To test this code, I used the following code:

```c#
using System;

public class CallingBadClass
{
  public static void Main()
  {
    CatchWithNoException();
    CatchExceptionAndWithNoException();
    CatchException();
  }

  private static void CatchExceptionAndWithNoException()
  {
    Console.Out.WriteLine("CatchExceptionAndWithNoException called.");
        
    try
    {
      Console.Out.WriteLine("In try block...");
      BadClass badClass = new BadClass();
      badClass.BadMethod();
      Console.Out.WriteLine("BadMethod called.");        
    }
    catch(Exception exception)
    {
      Console.Out.WriteLine("Catch entered - exception type is " +
      exception.GetType().FullName);                        
    }
    catch
    {
      Console.Out.WriteLine("Catch entered.");                        
    }
    finally
    {
      Console.Out.WriteLine("Finally entered.");
    }
  }
    
  private static void CatchException()
  {
    Console.Out.WriteLine("CatchException called.");
        
    try
    {
      Console.Out.WriteLine("In try block...");
      BadClass badClass = new BadClass();
      badClass.BadMethod();
      Console.Out.WriteLine("BadMethod called.");        
    }
    catch(Exception exception)
    {
      Console.Out.WriteLine("Catch entered - exception type is " +
        exception.GetType().FullName);                        
    }
    finally
    {
      Console.Out.WriteLine("Finally entered.");
    }
  }
    
  private static void CatchWithNoException()
  {
    Console.Out.WriteLine("CatchWithNoException called.");
        
    try
    {
      Console.Out.WriteLine("In try block...");
      BadClass badClass = new BadClass();
      badClass.BadMethod();
      Console.Out.WriteLine("BadMethod called.");        
    }
    catch
    {
      Console.Out.WriteLine("Catch entered.");                        
    }
    finally
    {
      Console.Out.WriteLine("Finally entered.");
    }
  }
}
```

Now, here's where it gets a bit tricky. There is a difference when you run this code using the 1.1 and 2.0 versions of the CLR. So when I compile the IL and C# code, I have to use the specific version of ilasm and csc (so if you try to compile the code, **make sure** you specify the directory that contains the version of ilasm and csc that you want to use). When I compile this code using the 1.1 tools, I get the following output:

```
CatchWithNoException called.
In try block...
Catch entered.
Finally entered.
CatchExceptionAndWithNoException called.
In try block...
Catch entered.
Finally entered.
CatchException called.
In try block...
```

At this point, the code stops and I get the "Common Language Runtime Debugging Services" window with the following message: "Application has generated an exception that could not be handled.". If I don't jump into the debugger, the program completes with this in the console window:

```
Unhandled Exception: System.Object
Finally entered.
```

Now, if I change to the 2.0 tools, the csc compiler gives me a warning that I didn't get before:

```
CallingBadClass.cs(28,9): warning CS1058: A previous catch clause already
        catches all exceptions. All non-exceptions thrown will be wrapped in a
        System.Runtime.CompilerServices.RuntimeWrappedException
```

Furthermore, when I run the program, I never get the dialog window, and the output is a bit different:

```
CatchWithNoException called.
In try block...
Catch entered.
Finally entered.
CatchExceptionAndWithNoException called.
In try block...
Catch entered - exception type is System.Runtime.CompilerServices.RuntimeWrappedException
Finally entered.
CatchException called.
In try block...
Catch entered - exception type is System.Runtime.CompilerServices.RuntimeWrappedException
Finally entered.
```

Note that now the `catch` block that doesn't specify an `Exception` to catch is worthless because the runtime wraps anything that is thrown and does not derive from `Exception` into a `RuntimeWrappedException`. The thrown object is availabe via the `WrappedException` property.

So if you're writing code in 2.0, then I agree with the recommendation. Since you should never swallow exceptions (you should at least always log them) and 2.0 now handles exceptions that aren't exceptions in a "graceful" fashion, there's no reason to use the `catch { }` technique. However, before 2.0 this option isn't available to you, and if you get a non-`Exception` the user experience is not very nice. If you're in 1.1 and you stumble across a method that is throwing a non-`Exception`, you may need to use the `catch { }` technique. This situation is so extrememly rare I wouldn't sweat it, but at least you know that it could happen!

> Published: 06.13.2006 07:35:56 PM CST