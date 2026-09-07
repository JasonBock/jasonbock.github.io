---
title: Virtual Properties and Sealed Classes in C# and VB .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Virtual Properties and Sealed Classes in C# and VB .NET

Somebody I was IMing with yesterday sent me this interesting problem. I still don't understand why the VB .NET compiler is acting the way it is (.NET 1.1) - maybe you can shed some light on it.

Consider the following classes in C#:

```c#
public class BaseClass
{
  protected virtual int IntegerData
  {
    get
    {
      return 42;
    }   
  }

  protected virtual void Method() {}
}

public sealed class DerivedClass : BaseClass
{
  protected override int IntegerData
  {
    get
    {
      return 44;
    }
  }

  protected override void Method() 
  {
    base.Method();
  }
}
```

This compiles with no errors. Now take a look at the comparable code in VB .NET:

```vb
Public Class BaseClass
  Protected Overridable ReadOnly Property IntegerData() As Integer
    Get
      Return 42
    End Get
  End Property

  Protected Overridable Sub Method()

  End Sub
End Class

Public NotInheritable Class DerivedClass
  Inherits BaseClass
  Protected Overrides ReadOnly Property IntegerData() As Integer
    Get
      Return 44
    End Get
  End Property

  Protected Overrides Sub Method()
    MyBase.Method()
  End Sub
End Class
```

This fails with the following errors: "'NotInheritable' classes cannot have members declared 'Protected'.", "'Public Overrides ReadOnly Property IntegerData() As Integer' cannot override 'Protected Overridable ReadOnly Property IntegerData() As Integer' because they have different access levels."

There's two things I don't get with this. First, why can't I override the property in VB .NET, but it's OK in C#? [1] Second, why is it OK to override the method in VB .NET, but not the property? This **feels** like a bug to me. Anybody have any other insights?

[1] Assume that you **can't** change the base class implementation. In other words, assume the base class is coming from an assembly and you don't have the source code (and don't try to get cute and say you can disassemble it and monkey with the CIL - just play fair and assume the base class cannot be altered).

> Published: 06.17.2004 08:46:35 AM CST