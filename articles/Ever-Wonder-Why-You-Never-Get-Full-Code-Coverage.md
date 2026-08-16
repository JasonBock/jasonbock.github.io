---
title: Ever Wonder Why You Never Get Full Code Coverage?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Ever Wonder Why You Never Get Full Code Coverage?

Take a look at this simple class:

```c#
namespace ForEachCoverage
{
  public static class EnumerateStuff
  {
    public static void PrintStuff(List<string> stuff)
    {
      foreach(var thing in stuff)
      {
        Console.Out.WriteLine(thing);
      }
    }

    public static string GetConventions(CallingConventions callingConvention)
    {
      var conventions = new List<string>();

      foreach(CallingConventions convention in 
        Enum.GetValues(typeof(CallingConventions)))
      {
        if((callingConvention & convention) == convention)
        {
          conventions.Add(convention.ToString()
            .ToLower(CultureInfo.CurrentCulture));
        }
      }

      return string.Join(" ", conventions.ToArray());
    }
  }
}
```

One method is enumerating over a list of strings, and the other is enumerating a list of `CallingConventions` values.

Here's my test class:

```c#
[TestClass]
public sealed class EnumerateStuffTests
{
  [TestMethod]
  public void EnumerateStuff()
  {
    var stuff = new List<string>() { "this", "that", "other" };
    ForEachCoverage.EnumerateStuff.PrintStuff(stuff);
  }

  [TestMethod]
  public void GetConventions()
  {
    ForEachCoverage.EnumerateStuff.GetConventions(
      CallingConventions.Any | CallingConventions.HasThis);
  }
}
```

OK, I'm really not testing anything of value, but what I want to demonstrate is the code coverage numbers:

![Not Covered](https://jasonbock.net/images/Not-Covered.png "Not Covered")

Hmmmm...it says I'm missing 2 blocks in `GetConventions()`. Let's pull up the pretty code highlighting:

![Code Coverage Highlighting](https://jasonbock.net/images/Code-Coverage-Highlighting.png "Code Coverage Highlighting")

D'oh! All of the lines are showing coverage! What's the deal?

The devil's in the compiler details. Take a look at the two methods in IL:

```
.class public abstract auto ansi sealed beforefieldinit 
  ForEachCoverageInIL.EnumerateStuff
  extends [mscorlib]System.Object
{
  .method public hidebysig static void  
    PrintStuff(class [mscorlib]System.Collections.Generic.List`1<string> stuff) cil managed
  {
    .maxstack  2
    .locals init ([0] string thing,
      [1] valuetype [mscorlib]System.Collections.Generic.List`1/Enumerator<string> CS$5$0000)
    IL_0000:  ldarg.0
    IL_0001:  callvirt instance valuetype 
      [mscorlib]System.Collections.Generic.List`1/Enumerator<!0> 
      class [mscorlib]System.Collections.Generic.List`1<string>::GetEnumerator()
    IL_0006:  stloc.1
    .try
    {
      IL_0007:  br.s       IL_001c

      IL_0009:  ldloca.s   CS$5$0000
      IL_000b:  call       instance !0 valuetype 
        [mscorlib]System.Collections.Generic.List`1/Enumerator<string>::get_Current()
      IL_0010:  stloc.0
      IL_0011:  call       class 
        [mscorlib]System.IO.TextWriter [mscorlib]System.Console::get_Out()
      IL_0016:  ldloc.0
      IL_0017:  callvirt   instance void [mscorlib]System.IO.TextWriter::WriteLine(string)
      IL_001c:  ldloca.s   CS$5$0000
      IL_001e:  call       instance bool valuetype 
        [mscorlib]System.Collections.Generic.List`1/Enumerator<string>::MoveNext()
      IL_0023:  brtrue.s   IL_0009
      IL_0025:  leave.s    IL_0035
    }
    finally
    {
      IL_0027:  ldloca.s   CS$5$0000
      IL_0029:  constrained. valuetype 
        [mscorlib]System.Collections.Generic.List`1/Enumerator<string>
      IL_002f:  callvirt   instance void [mscorlib]System.IDisposable::Dispose()
      IL_0034:  endfinally
    }
    IL_0035:  ret
  }

  .method public hidebysig static string GetConventions(
    valuetype [mscorlib]System.Reflection.CallingConventions callingConvention) cil managed
  {
    .maxstack  3
    .locals init ([0] class [mscorlib]System.Collections.Generic.List`1<string> conventions,
             [1] valuetype [mscorlib]System.Reflection.CallingConventions convention,
             [2] class [mscorlib]System.Collections.IEnumerator CS$5$0000,
             [3] class [mscorlib]System.IDisposable CS$0$0001)
    IL_0000:  newobj     instance void class 
      [mscorlib]System.Collections.Generic.List`1<string>::.ctor()
    IL_0005:  stloc.0
    IL_0006:  ldtoken    [mscorlib]System.Reflection.CallingConventions
    IL_000b:  call       class [mscorlib]System.Type 
      [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)
    IL_0010:  call       class [mscorlib]System.Array 
      [mscorlib]System.Enum::GetValues(class [mscorlib]System.Type)
    IL_0015:  callvirt   instance class 
      [mscorlib]System.Collections.IEnumerator [mscorlib]System.Array::GetEnumerator()
    IL_001a:  stloc.2
    .try
    {
      IL_001b:  br.s       IL_004a

      IL_001d:  ldloc.2
      IL_001e:  callvirt   instance object 
        [mscorlib]System.Collections.IEnumerator::get_Current()
      IL_0023:  unbox.any  [mscorlib]System.Reflection.CallingConventions
      IL_0028:  stloc.1
      IL_0029:  ldarg.0
      IL_002a:  ldloc.1
      IL_002b:  and
      IL_002c:  ldloc.1
      IL_002d:  bne.un.s   IL_004a
      IL_002f:  ldloc.0
      IL_0030:  ldloc.1
      IL_0031:  box        [mscorlib]System.Reflection.CallingConventions
      IL_0036:  callvirt   instance string [mscorlib]System.Object::ToString()
      IL_003b:  call       class [mscorlib]System.Globalization.CultureInfo 
        [mscorlib]System.Globalization.CultureInfo::get_CurrentCulture()
      IL_0040:  callvirt   instance string [mscorlib]System.String::ToLower(
        class [mscorlib]System.Globalization.CultureInfo)
      IL_0045:  callvirt   instance void class 
        [mscorlib]System.Collections.Generic.List`1<string>::Add(!0)
      IL_004a:  ldloc.2
      IL_004b:  callvirt   instance bool 
        [mscorlib]System.Collections.IEnumerator::MoveNext()
      IL_0050:  brtrue.s   IL_001d
      IL_0052:  leave.s    IL_0065
    } 
    finally
    {
      IL_0054:  ldloc.2
      IL_0055:  isinst     [mscorlib]System.IDisposable
      IL_005a:  stloc.3
      IL_005b:  ldloc.3
      IL_005c:  brfalse.s  IL_0064
      IL_005e:  ldloc.3
      IL_005f:  callvirt   instance void [mscorlib]System.IDisposable::Dispose()
      IL_0064:  endfinally
    }
    IL_0065:  ldstr      " "
    IL_006a:  ldloc.0
    IL_006b:  callvirt   instance !0[] class 
      [mscorlib]System.Collections.Generic.List`1<string>::ToArray()
    IL_0070:  call       string [mscorlib]System.String::Join(string,
                                                              string[])
    IL_0075:  ret
  }
}
```

That's a lot of code, so let's break it down into the key parts. In `PrintStuff()` the thing you're actually enumerating over is a nested type in `List<T>`, called `Enumerator<T>` (that's what's stored in `CS$5$0000`). The `foreach` keyword expands into a `try ... finally` block, and in the `finally` block, the enumerator's `Dispose()` method is called.

Now, the same thing (more or less) happens in `GetConventions()`, but there's a key difference. The `finally` block doesn't seem to assume it can blindly call `Dispose()` on the object stored in `CS$5$0000`, which in this method is an `IEnumerator`-based object. Sure, `IEnumerator` doesn't implement `IDisposable`, but that doesn't necessarily mean that what's stored in the object reference doesn't. Therefore, the compiler does a check to ensure the object implements `IDisposable` (the code at `IL_0055`), and if it doesn't, the `brfalse.s` line at `IL_005c` moves you down to `IL_0064`. Therefore, the 2 lines of code, `IL_005e` and `IL_005f` will never get executed no matter what you do in your tests. At least I can't think of anything that would force these 2 blocks to be executed.

So, if you struggling to get to 100% in your tests and you're just a tenth or a hundredth of a point off from that magical number, just remember that there may be nothing you can do to hit that number.

One interesting thing: if I call the IL version of `GetConventions()`, VS reports that only 1 block of code isn't executed, but if I step through the IL in the debugger, I clearly see that the 2 lines of code in the finally block aren't executed. I have no idea why VS reports only 1 in this case.

> Published: 10.30.2008 02:06:16 PM CST