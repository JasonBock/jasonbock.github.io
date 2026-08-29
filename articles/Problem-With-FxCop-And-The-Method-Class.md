---
title: Problem With FxCop and the Method Class
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Problem With FxCop and the Method Class

Last night I was writing some FxCop rules to demonstrate to the client, and I ran into an odd problem trying to transform a `MethodInfo` object into a `Method` object via `Method.GetMethod()`. I've uploaded the code that demonstrates the problem, which you can get here - here's a quick summary of the issue. I defined a class with two methods:

```c#
public sealed class CustomerMethods
{
  public CustomerMethods() : base() {}

  public void SimpleMethod() {}

  public GetCustomerResponse GetCustomer(GetCustomerRequest request)
  {
    return null;
  }
}
```

Then I wrote two NUnit tests that show what the problem is:

```c#
[TestFixture]
public sealed class MethodProblemTests
{
  [Test]
  public void InspectGetCustomer()
  {
    Type typeToTest = typeof(CustomerMethods);
    MethodInfo methodInfo = typeToTest.GetMethod("GetCustomer");
    Assert.IsNotNull(methodInfo, "The MethodInfo object is null.");
    Method method = Method.GetMethod(methodInfo);
    Assert.IsNotNull(method, "The Method object is null.");
  }

  [Test]
  public void InspectSimpleMethod()
  {
    Type typeToTest = typeof(CustomerMethods);
    MethodInfo methodInfo = typeToTest.GetMethod("SimpleMethod");
    Assert.IsNotNull(methodInfo, "The MethodInfo object is null.");
    Method method = Method.GetMethod(methodInfo);
    Assert.IsNotNull(method, "The Method object is null.");
  }
}
```

`InspectSimpleMethod()` succeeded, but `InspectGetCustomer()` fails on the second `Assert.IsNotNull()`. No matter what I do when I get the `MethodInfo` object via `GetMethod()`, I always get a null `Method` reference. But this only happens with the method that has a parameter; `SimpleMethod()` takes no arguments and returns void and I can get a `Method` object for the simple case.

I haven't done any Refelctor digging into the FxCop assemblies. I didn't look yet at FxCop's message boards to see if this is a bug. I also noticed that there's a new release candidate for FxCop (1.32, posted March 29th). This may fix the problem but I haven't checked it out yet. The weird thing is that I didn't see this when I was developing my extensible .NET compiler, but looking back on that code most of my test methods took no arguments! However, there are some that have arguments and they seem fine.

If you have any insights as to why this isn't working, feel free to post your ideas in the comments area.

> Published: 05.19.2005 04:34:12 PM CST