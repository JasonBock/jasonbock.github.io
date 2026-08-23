---
title: MSTest and `TestContext`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# MSTest and `TestContext`

I don't know if I've mentioned this issue in a previous blog entry ... but it always annoys me (ever so slightly) that I have to talk about it.

There's a `TestContext` class that you can use in your unit tests that you can use to log information to the test output, find out where the test directory is, etc. But to use it, you have to do something that I find very odd:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace Testing
{
  [TestClass]
  public class BunchOfTests
  {
    private TestContext context;

    [TestMethod]
    public void TestSomeCode() { }
        
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

What I find odd is that there's no `TestContextAttribute` class to mark the property. You have to name the property "TestContext" - this smells like the JUnit days of old where you had to start every test method with the text, "test". Ugh! I don't understand why Microsoft didn't do this:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace Testing
{
  [TestClass]
  public class BunchOfTests
  {
    private TestContext context;

    [TestMethod]
    public void TestSomeCode() { }
  
    // NOTE: This is invalid code - TestContextAttribute doesn't exist!
    [TestContext]      
    public TestContext Context
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

This completely frees me to name the property anything I want, so long as it gets and sets an object of type `TestContext`. I mean, you can do this with your class and method names since they're marked with attributes - why not do this with the `TestContext` property?

It's not a big deal, but it seems inconsistent with the rest of the unit testing framework. I don't see a technical reason why this couldn't be done. Maybe this will be added in a future version (I haven't look at the Orcas world yet).

> Published: 02.16.2007 08:29:52 AM CST