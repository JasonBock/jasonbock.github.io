---
title: More Attribute Description Information
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# More Attribute Description Information

OK, here's the crux of the description problem. Here's the CIL from the `ServiceProcessDescriptionAttribute`'s constructor:

```
.method public hidebysig specialname rtspecialname
  instance void  .ctor(string description) cil managed
{
  // Code size       15 (0xf)
  .maxstack  8
  IL_0000:  ldarg.0
  IL_0001:  ldc.i4.0
  IL_0002:  stfld      bool System.ServiceProcess.ServiceProcessDescriptionAttribute::replaced
  IL_0007:  ldarg.0
  IL_0008:  ldarg.1
  IL_0009:  call       instance void [System]System.ComponentModel.DescriptionAttribute::.ctor(string)
  IL_000e:  ret
} // end of method ServiceProcessDescriptionAttribute::.ctor
```

I'm sure I just violated something by doing this, but hey, it's only a couple lines of code! Note that at label `IL_0009` it's calling the base class's constructor, which looks like this:

```
.method public hidebysig specialname rtspecialname
  instance void  .ctor(string description) cil managed
{
  // Code size       14 (0xe)
  .maxstack  8
  IL_0000:  ldarg.0
  IL_0001:  call       instance void [mscorlib]System.Attribute::.ctor()
  IL_0006:  ldarg.0
  IL_0007:  ldarg.1
  IL_0008:  stfld      string System.ComponentModel.DescriptionAttribute::description
  IL_000d:  ret
} // end of method DescriptionAttribute::.ctor
```

So, unless my eyes are deceiving me (which is definitely a possibility), the field called `description` in `DescriptionAttribute` should be equal to the constructor's value. Right? But it's not - here's what VS .NET shows in the *Locals* window:

![Locals Window](https://jasonbock.net/images/Locals-Window.png "Locals Window")

But, if I subclass `DescriptionAttribute` myself like this:

```c#
public class MyServiceDescriptionAttribute : DescriptionAttribute
{
  bool m_Val;

  public MyServiceDescriptionAttribute(string Desc) : base(Desc)
  {
    m_Val = false;
  }

  public override string Description
  {
    get
    {
      return base.Description;
    }
  }
}
```

It works just perfectly. And check out the CIL for my constructor:

```
.method public hidebysig specialname rtspecialname
  instance void  .ctor(string Desc) cil managed
{
  // Code size       15 (0xf)
  .maxstack  2
  IL_0000:  ldarg.0
  IL_0001:  ldarg.1
  IL_0002:  call       instance void [System]System.ComponentModel.DescriptionAttribute::.ctor(string)
  IL_0007:  ldarg.0
  IL_0008:  ldc.i4.0
  IL_0009:  stfld      bool DescriptionAttributeTest.MyServiceDescriptionAttribute::m_Val
  IL_000e:  ret
} // end of method MyServiceDescriptionAttribute::.ctor
```

Other than the fact that my code calls the base class's constructor first and then does the boolean field assignment (the order is flipped in `ServiceProcessDescriptionAttribute`), nothing looks different to me.

Argh!

> Published: 07.22.2002 12:35:00 PM CST