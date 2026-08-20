---
title: Unit Tests Code Generation in VS 2008 and Automatic Properties
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unit Tests Code Generation in VS 2008 and Automatic Properties

Here's an interesting quirk. In VS 2008, when you right-click on a test project and add a new test class, it generates code that automatically contains the `TestContext` property (which you have to manually do in VS 2005). But what's weird is that they don't use the automatic property syntax in C# 3.0. Here's what you get once the file is created:

```c#
[TestClass]
public class UnitTest1
{
  public UnitTest1()
  {
    //
    // TODO: Add constructor logic here
    //
  }

  private TestContext testContextInstance;

  /// <summary>
  ///Gets or sets the test context which provides
  ///information about and functionality for the current test run.
  ///</summary>
  public TestContext TestContext
  {
    get
    {
      return testContextInstance;
    }
    set
    {
      testContextInstance = value;
    }
  }
```

If I change the code to this:

```c#
[TestClass]
public class UnitTest1
{
  public UnitTest1()
  {
    //
    // TODO: Add constructor logic here
    //
  }

  /// <summary>
  ///Gets or sets the test context which provides
  ///information about and functionality for the current test run.
  ///</summary>
  public TestContext TestContext
  {
    get;
    set;
  }
```

things work just as well, and that's a lot less code.

Odd that the IDE doesn't generate code using the new language features ... unless there's a switch somewhere that isn't set that I'm missing ...

> Published: 12.07.2007 04:10:02 PM CST