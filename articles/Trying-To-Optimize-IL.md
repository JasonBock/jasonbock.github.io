---
title: Trying To Optimize IL
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Trying To Optimize IL

One of my (twisted) hobbies is playing with IL. I don't do it as much as I used to, but I still have fun playing with words like "ldstr" and "callvirt". Anyway, here's a fun digression into reducing the opcode size of a method and if it's really worth it.

Let's say I have the following class:

```c#
public sealed class Person
{
  public string FirstName
  {
    get;
    set;
  }

  public string LastName
  {
    get;
    set;
  }

  public override string ToString()
  {
    StringBuilder toString = new StringBuilder();
        
    toString.Append("First Name: ").Append(this.FirstName)
      .Append(", Last Name: ").Append(this.LastName);
        
    return toString.ToString();
  }
}
```

Now if I bust the `ToString()` method open to see the IL (using Reflector), I get this:

```
.method public hidebysig virtual instance string ToString() cil managed
{
  .maxstack 2
  .locals init (
    [0] class [mscorlib]System.Text.StringBuilder toString,
    [1] string CS$1$0000)
  L_0000: nop 
  L_0001: newobj instance void [mscorlib]System.Text.StringBuilder::.ctor()
  L_0006: stloc.0 
  L_0007: ldloc.0 
  L_0008: ldstr "First Name: "
  L_000d: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0012: ldarg.0 
  L_0013: call instance string TestingGrounds.Person::get_FirstName()
  L_0018: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_001d: ldstr ", Last Name: "
  L_0022: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0027: ldarg.0 
  L_0028: call instance string TestingGrounds.Person::get_LastName()
  L_002d: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0032: pop 
  L_0033: ldloc.0 
  L_0034: callvirt instance string [mscorlib]System.Object::ToString()
  L_0039: stloc.1 
  L_003a: br.s L_003c
  L_003c: ldloc.1 
  L_003d: ret 
}
```
Note that the this was a debug build. If I go to a release build, nothing changes, but if I turn optimizations on, the IL changes slightly:

```
.method public hidebysig virtual instance string ToString() cil managed
{
  .maxstack 2
  .locals init (
    [0] class [mscorlib]System.Text.StringBuilder toString)
  L_0000: newobj instance void [mscorlib]System.Text.StringBuilder::.ctor()
  L_0005: stloc.0 
  L_0006: ldloc.0 
  L_0007: ldstr "First Name: "
  L_000c: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0011: ldarg.0 
  L_0012: call instance string TestingGrounds.Person::get_FirstName()
  L_0017: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_001c: ldstr ", Last Name: "
  L_0021: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0026: ldarg.0 
  L_0027: call instance string TestingGrounds.Person::get_LastName()
  L_002c: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0031: pop 
  L_0032: ldloc.0 
  L_0033: callvirt instance string [mscorlib]System.Object::ToString()
  L_0038: ret 
}
```

The 2nd local variable isn't used anymore to store the final `StringBuilder` result. The `nop` is also gone. But can we do better?

```
.method public hidebysig virtual string ToString() cil managed
{
  .maxstack 2
  L_0000: newobj instance void [mscorlib]System.Text.StringBuilder::.ctor()
  L_0007: ldstr "First Name: "
  L_000c: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0011: ldarg.0 
  L_0012: call instance string PersonIL.Person::get_FirstName()
  L_0017: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_001c: ldstr ", Last Name: "
  L_0021: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0026: ldarg.0 
  L_0027: call instance string PersonIL.Person::get_LastName()
  L_002c: callvirt instance class [mscorlib]System.Text.StringBuilder [mscorlib]System.Text.StringBuilder::Append(string)
  L_0033: callvirt instance string [mscorlib]System.Object::ToString()
  L_0038: ret 
}
```

Yes. There really isn't a need to store the `StringBuilder`; we can just keep it on the stack. In this final example, we've removed the need to reload the local variable onto the stack (`ldloc.0`).

But ... does any of this matter? I've run a number of tests, and as far as I can tell, the execution time between these three examples is identical. My point is I've heard some people say that they want inline IL in C# or VB. I used to think that would be a nice addition, but I have the feeling that the runtime is doing some optimizations that would nullify any optimizations anyone thinks they could do over what the managed compilers pull of. Maybe there's another instance that someone's run into, though, where the compilers produced some really crappy IL and a hand-optimization would squeeze out some performance. There's also some things you can do at the IL layer that you can't in VB or C#, so having that "out" may be a nice thing to have.

I still want an IL-based VS project type, though ...

> Published: 01.25.2008 12:53:20 PM CST