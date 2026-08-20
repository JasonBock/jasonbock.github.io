---
title: Tests and Disposable Fields
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Tests and Disposable Fields

Lately we had a big push at the client to incorporate Code Analysis in the build process. We've ignored some of the rules [1], but we have the majority of them on, and it's been good to find things in the code that needed to be changed. However, there was one situation where Code Analysis suggested one thing, but doing it really didn't help (and made things a litle worse).

Here's the scenario. Say you have a test class like this:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace DisposingInTests
{
  [TestClass]
  public sealed class TestsWithDisposableField
  {
    private TestContext context;
        
    [TestInitialize]
    public void TestInitialize()
    {
      this.context.WriteLine("TestInitialize");
    }

    [TestCleanup]
    public void TestCleanup()
    {
      this.context.WriteLine("TestCleanup");
    }
        
    [TestMethod]
    public void Run()
    {
      this.context.WriteLine("Run");
      int x = 3;
      Assert.AreEqual(3, x);
    }

    public TestContext TestContext
    {
      get
      {
        return this.context;
      }
      set
      {
        this.context = value;
      }
    }
  }
}
```

If I run the `Run` test, I get the following output:

```
TestInitialize
Run
TestCleanup
```

No surprises there. Now, let's say I add the following class to my code base:

```c#
using System;

namespace DisposingInTests
{
  public sealed class Disposable : IDisposable
  {
    public void Dispose()
    {
    }
  }
}
```

OK, this is definitely not the way to implement `IDisposable`, but the point is that I now have an object that is disposable. Let's add this as a field to my test class:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace DisposingInTests
{
  [TestClass]
  public sealed class TestsWithDisposableField
  {
    private TestContext context;
    private Disposable disposable;
        
    [TestInitialize]
    public void TestInitialize()
    {
      this.context.WriteLine("TestInitialize");
      this.disposable = new Disposable();
    }

    [TestCleanup]
    public void TestCleanup()
    {
      this.context.WriteLine("TestCleanup");

      if(this.disposable != null)
      {
        this.disposable.Dispose();            
      }
    }
        
    [TestMethod]
    public void Run()
    {
      this.context.WriteLine("Run");
      int x = 3;
      Assert.AreEqual(3, x);
    }

    public TestContext TestContext
    {
      get
      {
        return this.context;
      }
      set
      {
        this.context = value;
      }
    }
  }
}
```

Again, this is pretty basic. But now ... Code Analysis rears its ugly head:

```
Error 1 CA1001 : Microsoft.Design : Implement IDisposable on 'DisposingInTests.TestsWithDisposableField' as it instantiates members of the following IDisposable types: DisposingInTests.Disposable C:\JasonBock\Personal\.NET Projects\DisposingInTests\TestsWithDisposableField.cs 7 DisposingInTests
```

OK, fine - I'll implement `IDisposable`:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace DisposingInTests
{
  [TestClass]
  public sealed class TestsWithDisposableField : IDisposable
  {
    private TestContext context;
    private Disposable disposable;
        
    [TestInitialize]
    public void TestInitialize()
    {
      this.context.WriteLine("TestInitialize");
      this.disposable = new Disposable();
    }

    [TestCleanup]
    public void TestCleanup()
    {
      this.context.WriteLine("TestCleanup");

      if(this.disposable != null)
      {
        this.disposable.Dispose();            
      }
    }
        
    [TestMethod]
    public void Run()
    {
      this.context.WriteLine("Run");
      int x = 3;
      Assert.AreEqual(3, x);
    }

    public TestContext TestContext
    {
      get
      {
        return this.context;
      }
      set
      {
        this.context = value;
      }
    }
        
    public void Dispose()
    {
      this.context.WriteLine("Dispose");
      this.TestCleanup();
    }
  }
}
```

But when I run `Run`, look at the printout:

```
TestInitialize
Run
TestCleanup
```

Even though I've implemented `IDisposable`, the test engine doesn't seem to care and never calls `Dispose()`.

Here's my point. It's not that uncommon to have a field in a test class that is `IDisposable`. A stream, a database connection ... there's lot of things that I declare as an instance field that gets initialized for every test in that class. The right thing to do from a .NET standpoint is to implement `IDisposable` and dispose of my instance fields. But in the MSTest world ... it doesn't help. In fact, if you rely on `Dispose()` getting called, you may end up leaking resources. Therefore, the best thing to do in this case is to suppress the rule:

```c#
[System.Diagnostics.CodeAnalysis.SuppressMessage(
  "Microsoft.Design", "CA1001:TypesThatOwnDisposableFieldsShouldBeDisposable"), TestClass]
public sealed class TestsWithDisposableField
```

Now, maybe in the future, MSTest will start handing test classes that implement `IDisposable` correctly, so one could make the argument that both approaches (`IDisposable` and `TestCleanup`) should be used in concert. There's definitely some validity there, just keep in mind that you have to go with both approaches for now and you can't rely on `Dispose()` being called.

[1] Like globalization, mobility, and the "consider strong-naming" rule.

> Published: 09.18.2007 08:32:38 PM CST