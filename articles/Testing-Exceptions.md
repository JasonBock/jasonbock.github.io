---
title: Testing Exceptions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Testing Exceptions

I ran across some code I wrote that goes through some basic tests for any custom `Exception`. Here you go:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.IO;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Binary;

namespace YourNamespaceGoesHere
{
  public abstract class ExceptionTests<T, TInner> : TestsBase 
    where T : Exception, new()
    where TInner : Exception, new()
  {
    private string message;
        
    protected ExceptionTests(string message) : base()
    {
      this.message = message;
    }
        
    protected void CreateExceptionTest()
    {
      T exception = new T();
      Assert.IsNotNull(exception.Message);
      Assert.IsNull(exception.InnerException);
    }

    protected void CreateExceptionWithMessageTest()
    {
      T exception = Activator.CreateInstance(typeof(T), this.message) as T;
      Assert.AreEqual(this.message, exception.Message);
      Assert.IsNull(exception.InnerException);
    }

    protected void CreateExceptionWithMessageAndInnerExceptionTest()
    {
      TInner innerException = new TInner();

      T exception = Activator.CreateInstance(typeof(T), this.message, innerException) as T;
      Assert.AreEqual(this.message, exception.Message);
      Assert.AreEqual(innerException, exception.InnerException);
    }

    protected void RoundtripExceptionTest()
    {
      T exception = Activator.CreateInstance(typeof(T), this.message) as T;
      T newException = null;

      IFormatter formatter = new BinaryFormatter();

      using(Stream stream = new MemoryStream())
      {
        formatter.Serialize(stream, exception);
        stream.Position = 0;
        newException = formatter.Deserialize(stream) as T;
      }

      Assert.IsNotNull(newException);
      Assert.AreEqual(this.message, newException.Message);
    }
  }
}
```

To use it, you'd do something like this:

```c#
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.IO;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Binary;

namespace YourNamespaceGoesHere
{
  [TestClass]
  public sealed class InvalidMoveExceptionTests : ExceptionTests<InvalidMoveException, ArgumentException>
  {
    private const string Message = "What a bad move!";

    public InvalidMoveExceptionTests()
      : base(InvalidMoveExceptionTests.Message)
    {
    }

    [TestMethod]
    public void CreateException()
    {
      this.CreateExceptionTest();
    }

    [TestMethod]
    public void CreateExceptionWithMessage()
    {
      this.CreateExceptionWithMessageTest();
    }

    [TestMethod]
    public void CreateExceptionWithMessageAndInnerException()
    {
      this.CreateExceptionWithMessageAndInnerExceptionTest();
    }

    [TestMethod]
    public void RoundtripException()
    {
      this.RoundTripExceptionTest();
    }
  }
}
```

Granted, you have to create a subclass to test your custom `Exception` type, but once you've done it, you can easily copy-and-paste the code. There may be a better way to do this, but this makes this a bit easier to test the key parts of an `Exception`. Enjoy!

> Published: 12.20.2007 06:24:05 PM CST