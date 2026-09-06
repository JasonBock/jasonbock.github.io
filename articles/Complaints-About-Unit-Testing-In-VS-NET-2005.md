---
title: Complaints About Unit Testing in VS .NET 2005
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Complaints About Unit Testing in VS .NET 2005

After playing with NUnit ... er, the unit testing framework in VS .NET 2005 ;), I've come across some annoyances. To illustrate them, I'll walk through an example.

What I wanted to do is have an abstract base class that would define a standard approach to testing my files:

```c#
namespace FileProcessing.Tests
{
  [TestClass()]
  public abstract class FileTestUnit
  {
    protected string testFile = string.Format("{0}.xml", Guid.NewGuid().ToString("D"));

    protected FileTestUnit() : base() {}

    [TestInitialize()]
    public void Initialize()
    {
      this.DeleteTestFile();
      this.OnInitialize();
    }

    private void DeleteTestFile()
    {
      if (File.Exists(this.testFile) == true)
      {
        File.Delete(this.testFile);
      }
    }

    protected abstract void OnInitialize();

    [TestCleanup()]
    public void Cleanup()
    {
      this.DeleteTestFile();
    }
  }
}
```

Then I'd have a bunch of test classes to ensure a bunch of file processing rules I have:

```c#
namespace FileProcessing.Tests
{
  [TestClass()]
  public class FileLoadingTestUnit : FileTestUnit
  {
    public FileLoadingTestUnit() : base() {}

    protected override void OnInitialize()
    {
      using (StreamWriter fileWriter = File.CreateText(this.thisFile))
      {
        //  Load the file...
      }
    }

    [TestMethod()]
    public void CheckFileLoading()
    {
      //  Check the file...
    }
  }
}
```

But this doesn't work - I get the following compilation warnings:

```
UTA003: TestClass attribute defined on abstract class FileProcessing.Tests.FileTestUnit
UTA005: Illegal use of attributes on FileProcessing.Tests.FileTestUnit.Initialize.The TestInitializeAttribute can be defined only inside a class marked with the TestClass attribute.
UTA006: Illegal use of attributes on FileProcessing.Tests.FileTestUnit.Cleanup. The TestCleanupAttribute can be defined only inside a class marked as TestClass.
```

The real issue that that `OnInitialize()` isn't being called in the subclass, which is a problem as the file is never made, so `CheckFileLoading()` fails. OK, fine, I'll make the class creatable by removing the abstract keyword (and subsequentially make `OnInitialize()` non-abstract) in the hope that this will fix it. But that doesn't help either - `OnInitialize()` **still** isn't called! These things just feel wrong to me. Why can't I define an abstract base testing class that has these initialize and cleanup methods that are called? Moreover, why can't I have base initialize and cleanup methods in a base class. This sucks, because now each specific test class has to have the same code in their own initialize and cleanup methods. I don't like duplicating code, and inheritance seems to be a good solution to solve my problem. But it adds new problems for reasons that don't seem correct.

I've blogged about this before, and I liked what NUnit did as it fit my expectations. I can only hope that VS .NET 2005 decides to change some of their restrictions in their testing framework. I wish I would've found this sooner as my guess is that it's too late to voice my opinion to the MS team about this, but one can try!

> Published: 10.26.2004 08:07:19 PM CST