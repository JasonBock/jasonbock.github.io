---
title: Unit Test, Setup and Teardown, and Inheritance ... Who's Right
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unit Test, Setup and Teardown, and Inheritance ... Who's Right

A while back, I was on a project where it looked like inheritance would help us reduce our unit test code base for a couple of scenarios. However, we ran into some oddities with setup and teardown methods, but we never got around to isolating it as we had other, more important stuff to address. Well, I had a couple of minutes over lunch to do some investigations, and this is what I came up with.

Let's say you had the following code:

```c#
using System;
using NUnit.Framework;

namespace UnitTestInheritanceTest
{
  [TestFixture]
  public class BaseUnitTests
  {
    [SetUp]
    protected virtual void BaseSetup()
    {
      Console.WriteLine("BaseUnitTests.BaseSetup invoked.");
    }

    [TearDown]
    protected virtual void BaseTeardown()
    {
      Console.WriteLine("BaseUnitTests.BaseTeardown invoked.");
    }
  }

  [TestFixture]
  public class SpecializedUnitTests : BaseUnitTests
  {
    [Test]
    public void MyTest()
    {
      Console.WriteLine("SpecializedUnitTests.MyTest invoked.");
    }
  }
}
```

Now, if I run `MyTest()` under NUnit (version 2.1), I get this:

```
BaseUnitTests.BaseSetup invoked.
SpecializedUnitTests.MyTest invoked.
BaseUnitTests.BaseTeardown invoked.
```

But if I run `MyTest()` under the NUnit Add-In (version 0.9.482d), the results are different:

```
------ Test started: Assembly: UnitTestInheritanceTest.dll ------

Out: SpecializedUnitTests.MyTest invoked.

1 succeeded, 0 failed, 0 skipped, took 0.02 seconds.
```

The key point is that NUnit runs both the setup and teardown methods from the base class, but the add-in doesn't. So ... who is right? Well, it depends. I haven't dug into the source code just yet, but here's some Reflection code that might shed some light on the problem:

```c#
[TestFixture]
public class ReflectionMethodTest
{
  [Test]
  public void ReflectBase()
  {
    Console.WriteLine("ReflectBase()");
    BaseUnitTests b = new BaseUnitTests();
    this.ReflectMethods(b);
  }

  [Test]
  public void ReflectSpecialized()
  {
    Console.WriteLine("ReflectSpecialized()");
    SpecializedUnitTests s = new SpecializedUnitTests();
    this.ReflectMethods(s);
  }

  private void ReflectMethods(object targetObject)
  {
    foreach(MethodInfo method in targetObject.GetType().GetMethods(
      BindingFlags.Instance |
      BindingFlags.Public | BindingFlags.NonPublic))
    {
      Console.WriteLine(method.ToString());
    }
  }
}
```

With these tests, you get the following results:

```
ReflectBase()
Void BaseTeardown()
Void BaseSetup()
Void Finalize()
Int32 GetHashCode()
Boolean Equals(System.Object)
System.String ToString()
System.Type GetType()
System.Object MemberwiseClone()

ReflectSpecialized()
Void BaseTeardown()
Void BaseSetup()
Void Finalize()
Int32 GetHashCode()
Boolean Equals(System.Object)
System.String ToString()
Void MyTest()
System.Type GetType()
System.Object MemberwiseClone()
```

However, if you change the code within the `foreach` enumeration like so:

```c#
[TestFixture]
public class ReflectionMethodTest
{
  [Test]
  public void ReflectBase()
  {
    Console.WriteLine("ReflectBase()");
    BaseUnitTests b = new BaseUnitTests();
    this.ReflectMethods(b);
  }

  [Test]
  public void ReflectSpecialized()
  {
    Console.WriteLine("ReflectSpecialized()");
    SpecializedUnitTests s = new SpecializedUnitTests();
    this.ReflectMethods(s);
  }

  private void ReflectMethods(object targetObject)
  {
    foreach(MethodInfo method in targetObject.GetType().GetMethods(
      BindingFlags.Instance |
      BindingFlags.Public | BindingFlags.NonPublic))
    {
      if(method.DeclaringType == targetObject.GetType())
      {
        Console.WriteLine(method.ToString());
      }
    }
  }
}
```

The results are quite different:

```
ReflectBase()
Void BaseTeardown()
Void BaseSetup()

ReflectSpecialized()
Void MyTest()
```

In other words, based on how you handle the list that comes back from `GetMethods()` can change your results significantly. It's possible that NUnit and NUnitAddIn handle the gathering of unit tests (and the related setup and teardown methods) in a different manner, but it would be nice if they were both consistent.

> Published: 07.26.2004 12:54:10 PM CST