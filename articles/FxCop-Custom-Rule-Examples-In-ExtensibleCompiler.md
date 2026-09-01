---
title: FxCop Custom Rule Examples in ExtensibleCompiler
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# FxCop Custom Rule Examples in ExtensibleCompiler

Late last week I posted my Extensible Compiler for .NET. To give you a feeling for how this works, I'll cover the two FxCop custom rules that come with code.

The first one is called *MustHandleUncheckedExceptionRule*. This basically checks to see if your code doesn't handle exceptions and does not mark the method with a `ThrowsAttribute`. Consider the following code snippet:

```c#
public class BadCheckedExceptions
{
  [Throws(typeof(ArgumentException))]
  private void BadMethod() {}

  public void CallMethod()
  {
    try
    {
      this.BadMethod();
    }
    catch(ApplicationException) {}
  }
}
```

`BadMethod()` states through the `ThrowsAttribute` that it may throw an exception of type `ArgumentException`. `CallMethod()` calls `BadMethod()`, but it doesn't catch `ArgumentException`, so the FxCop rule will create an error, which ends up looking like this in EC.exe:

```
EC compilation was unsuccessful.
Error 1
    code: FXC1000
    Description - Method BadCheckedExceptions.CallMethod():Void in class ExtensibleCompiler.Framework.Tests.BadCheckedExceptions must handle or state that it may throw an exception of type System.ArgumentException.
```

The other rule is called *MustInvokeBaseClassMethodRule*. This rule ensures that if you have a base class with an accessible virtual method, it must be the first method you call if you override it in a subclass and you have to call it. Again, here's a code snippet to illustrate:

```c#
public class BadInvokesBaseClass
{
  [MustInvoke()]
  public virtual void MustInvokeMethod() {}

  [MustInvoke()]
  public virtual void OptionalInvokeMethod() {}
}

public class BadMustInvokesSubClass : BadInvokesBaseClass
{
  public override void MustInvokeMethod() {}

  public override void OptionalInvokeMethod() 
  {
    this.MustInvokeMethod();
  }
}
```

The `MustInvokeAttribute` is used by a developer to force a subclass to invoke it. In the case of `BadMustInvokesSubClass`, `MustInvokeMethod()` is incorrect because it does not invoke the base class's implementation. `OptionalInvokeMethod()` is incorrect for the same reason. The error in EC.exe looks like this:

```
EC compilation was unsuccessful.
Error 1
    Code: FXC1002
    Description - Method BadMustInvokesSubClass.MustInvokeMethod():Void in class ExtensibleCompiler.Framework.Tests.BadMustInvokesSubClass must call the base class implementation first.
Error 2
    Code: FXC1002
    Description - Method BadMustInvokesSubClass.OptionalInvokeMethod():Void in class ExtensibleCompiler.Framework.Tests.BadMustInvokesSubClass must call the base class implementation first.
```

Note that I created another rule called `IncorrectMustInvokeUsageRule` that makes sure the `MustInvokeAttribute` is not used on static or non-virtual methods.

The checked exceptions rule was a lot harder to write, but it seems more pratical than the invocation rule. Even if you don't use the extensibility aspect to add your own rules at compile-time, hopefully you'll get something out of the FxCop code I wrote.

> Published: 03.21.2005 12:36:23 PM CST