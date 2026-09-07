---
title: Potential Pitfalls in Using FileGenerator as a Language Translator
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Potential Pitfalls in Using FileGenerator as a Language Translator

A week ago, I [blogged](https://jasonbock.net/articles/Finding-A-Use-For-FileGenerator-Language-Translation.html) that you could you my FileGenerator add-in for Reflector to translate your code from one .NET language to another. However, I should point out that the translation may not be perfect due to language mismatches. Here's an example to illustrate this point.

Let's say you had the following interfaces with a class implementing those interfaces in VB .NET:

```vb
Option Strict On
Option Explicit On 

Public Interface IOne
  Sub CallThis()
End Interface

Public Interface ITwo
  Sub CallThis()
End Interface

Public Interface IThree
  Sub CallThat()
End Interface

Public Interface IFour
  Sub CallTheOther()
End Interface

Public Class Implementor
  Implements IOne, ITwo, IThree, IFour

  Private Sub HandlesEverything() Implements IOne.CallThis, ITwo.CallThis, IThree.CallThat, IFour.CallTheOther

  End Sub
End Class
```

This is one feature of VB .NET that C# does not have in that you can have one method handle a number of interface definitions. So long as the parameters and return type are the same, it doesn't matter what the name of the interface method is. Unfortunately, C# doesn't do this, and when I translate this code via the Reflector+FileGenerator technique, you can see that the results yield code that does not compile:

```c#
public interface IOne
{
  // Methods
  void CallThis();
}

public interface ITwo
{
  // Methods
  void CallThis();
}

public interface IThree
{
  // Methods
  void CallThat();
}

public interface IFour
{
  // Methods
  void CallTheOther();
}

public class Implementor : IOne, ITwo, IThree, IFour
{
  // Methods
  public Implementor
  {
  }

  private void HandlesEverything()
  {
  }
}
```

If you try to compile this code, you'll get four errors stating that `Implementor` doesn't implement the methods in the four interfaces.

I hope this execise doesn't come across as a dig against Reflector or my add-in. This is just a natural fact that not all .NET languages are exactly the same. In this case, you'd have to fix the C# version of `Implementor` so it has the same semantics as the VB .NET version (which isn't that hard to do, but I'll leave it as an exercise for the interested reader ;) ). Therefore, keep this in mind if you have to translate your code from one language to another. You may have to do post-code-generation fixups to get the new code to compile.

> Published: 06.24.2004 10:01:58 AM CST