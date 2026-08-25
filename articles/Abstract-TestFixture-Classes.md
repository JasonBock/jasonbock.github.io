---
title: Abstract `TestFixture` Classes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Abstract `TestFixture` Classes

Wow, a technical post with code! I haven't done that in a while. I'm not feeling the hottest right now - I have a bad sore throat and I have a slight fever - but since I know my manager reads my blog I hope he sees this in case I take the day off tomorrow. Anyway, it has to do with unit tests in .NET. Here's the scenario. Let's say you have tests in a base class and you have two subclasses of that base class:

```c#
[TestFixture]
public abstract class BaseClass
{
  public BaseClass() : base() {}

  [Test]
  public void TestMethod() 
  {
    Trace.WriteLine(this.GetType().FullName);
  }
}

[TestFixture]
public class SubClassOne : BaseClass
{
  public SubClassOne() : base() {}
}

[TestFixture]
public class SubClassTwo : BaseClass
{
  public SubClassTwo() : base() {}
}
```

Now, note that `BaseClass` is `abstract`, so you really can't run `TestMethod()` as you can't make an instance of `BaseClass`. But TestDriven.NET will actually look for concrete implementations of the abstract class and run the method from those classes [1]. In other words, when I run `TestMethod()`, this is what I see in the Output window in VS .NET:

```
------ Test started: Assembly: Tests.dll ------

Tests.SubClassOne
Tests.SubClassTwo

2 succeeded, 0 failed, 0 skipped, took 0.01 seconds.



---------------------- Done ----------------------
```

The problem that I had at the client was that our base abstract class didn't have the `[TestFixture]` attribute on it. I can kind of understand why the developer (who also happens to be my manager :) ) did this as it shouldn't really matter that the base class isn't a test fixture; only the subclasses should have that metadata. But that's the way it works.

What I'd like to see in a future version of TestDriven.NET (and I've already let Jamie know about it) is that you would get a dynamic menu option in VS .NET's context menu. For example, right after the *Run Test(s)* option, there would be a *Run Test...* menu option that would then open a submenu that would list all of the possible test methods you could run. That may not be very performant, but it would be a nice option as I don't want to always run every subclass's implementation of the base class's test method.

Anyway, just another oddity I've fun - hopefully you can avoid it in the future.

[1] I didn't know this until tonight when Jamie Cansdale told me about it in a MSN conversation - thanks, Jamie!

> Published: 08.30.2005 07:24:10 PM CST