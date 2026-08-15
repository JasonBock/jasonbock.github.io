---
title: Code Injection With CCI - Part 1
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 1

Recently Microsoft released [CCI](https://github.com/microsoft/cci) on CodePlex. These are assemblies that allow you do all sorts of lower-level compiler-related things, like assembly generation and modification, ASTs, etc. I've seen CCI pop up here and there throughout the years (FxCop, Code Contracts, etc.) and now all of the source code is there for us to look at. This is a very cool move on Microsoft's part, although I personally think it's taken a bit too long to happen. Nevertheless, it's all there, and it's time to dig in and see how things work.

I decided to use the code I have for my "Reflection in .NET" talk where I modify an assembly based on the existence of attributes in the code. The code currently uses Cecil to do the modification, so I made new projects that ripped out the Cecil-based code and replaced it with CCI's functionality. Before I get into CCI, though, let me show you what my sample does starting with the code that's the target for modification:

```c#
using Customers;
using System;
using System.Reflection;
using System.Text;

namespace CciInjected
{
  public sealed class VerboseCustomers : Customer
  {
    private VerboseCustomers() 
      : base()
    {
    }

    public VerboseCustomers(Guid id, int age, string firstName, string lastName)
      : this()
    {
      var currentMethod = MethodBase.GetCurrentMethod().ToString();
      Console.Out.WriteLine(currentMethod + " started");

      if(firstName == null)
      {
        Console.WriteLine(currentMethod + " - exception was thrown");
        throw new ArgumentNullException("firstName");
      }

      if(lastName == null)
      {
        Console.WriteLine(currentMethod + " - exception was thrown");
        throw new ArgumentNullException("lastName");
      }

      this.Id = id;
      this.Age = age;
      this.FirstName = FirstName;
      this.LastName = lastName;
            
      Console.WriteLine(currentMethod + " finished");
    }

    public override string ToString()
    {
      return new StringBuilder()
        .Append("Age: ").Append(this.Age)
        .Append(Constants.Separator)
        .Append("Id: ").Append(this.Id)
        .Append(Constants.Separator)
        .Append("LastName: ").Append(this.LastName)
        .Append(Constants.Separator)
        .Append("FirstName: ").Append(this.FirstName).ToString();
    }
  }
}
```

There are three aspects that can be generalized and pulled out of this code such that they can be reused in other code bases. They are:

* Checking for parameters to not be null.
* Function tracing (enter, exit, and thrown exceptions)
* `ToString()` implementation

Granted, the last one may be a bit of a stretch, but all I'm doing in that method is concatenating the values of the public properties on that object, and that logic can easily be repeated on any object.

What I want to have is something like this:

```c#
using CciInjector.Attributes;
using Customers;
using System;

namespace CciInjected
{
  [ToString]
  public sealed class AttributedCustomer : Customer
  {
    private AttributedCustomer() 
      : base()
    {
    }

    [Trace]
    public AttributedCustomer(Guid id, [NotNull]int age,
      [NotNull]string firstName, [NotNull]string lastName, BirthDate birthDate)
      : this()
    {
      this.Id = id;
      this.Age = age;
      this.FirstName = firstName;
      this.LastName = lastName;
      this.BirthDate = birthDate;
    }

    public BirthDate BirthDate
    {
      get;
      set;
    }
  }
}
```

Of course, if this code was compiled with csc.exe, it wouldn't have those aspects as attributes are just metadata; something has to act upon the existence of those attributes and act accordingly.

This is where my demo code comes into play. I have a console application called CciInjected.exe:

```c#
using System;

namespace CciInjected
{
  class Program
  {
    static void Main(string[] args)
    {
      Program.CreateCustomer(
        Guid.NewGuid(), 22, "Joe", "Smith", 
        new BirthDate(new DateTime(1980, 1, 2)));
      Program.CreateCustomer(
        Guid.NewGuid(), 22, null, "Smith", 
        new BirthDate(new DateTime(1980, 1, 2)));
    }

    private static void CreateCustomer(Guid id, int age, string firstName, 
      string lastName, BirthDate birthDate)
    {
      try
      {
        Console.Out.WriteLine(new AttributedCustomer(
          id, age, firstName, lastName, birthDate).ToString());
      }
      catch(ArgumentNullException)
      {
        Console.Out.WriteLine("ArgumentNullException");
      }
    }
  }
}
```

Now, when this is compiled, CciInjected will produce the following output:

```
CciInjected.AttributedCustomer
CciInjected.AttributedCustomer
```

but when my injection process is performed on CciInjected.exe, the output changes:

```
Void .ctor(System.Guid, Int32, System.String, System.String, CciInjected.BirthDate) started
Void .ctor(System.Guid, Int32, System.String, System.String, CciInjected.BirthDate) finished
BirthDate: 1/2/1980 12:00:00 AM || Age: 22 || Id: 748bfa63-2213-4f9a-8e5d-e8820c24267e || FirstName: Joe || LastName: Smith
Void .ctor(System.Guid, Int32, System.String, System.String, CciInjected.BirthDate) started
Void .ctor(System.Guid, Int32, System.String, System.String, CciInjected.BirthDate) - exception was thrown
ArgumentNullException
Void .ctor(System.Guid, Int32, System.String, System.String, CciInjected.BirthDate) started
Void .ctor(System.Guid, Int32, System.String, System.String, CciInjected.BirthDate) finished
```

This is what `CciInjector` does. It takes the name of an assembly on the command line, and does all the modifications on the assembly based on attributes that exist on members anywhere within the assembly. In Part 2, I'll start to dive into the outer layer of `CciInjector`.

> Published: 04.22.2009 01:08:53 PM CST