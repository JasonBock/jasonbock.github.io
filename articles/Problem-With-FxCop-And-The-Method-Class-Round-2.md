---
title: Problem With FxCop and the Method Class, Round 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Problem With FxCop and the Method Class, Round 2

After digging into [this problem](https://jasonbock.net/articles/Problem-With-FxCop-And-The-Method-Class.html), more ... well, I don't have the full answers, but I'm getting closer!

First, now I know why my test for the extensible .NET compiler worked. While most of my test methods took no arguments, the few that did used types defined by mscorlib, like int, string, and Guid. FxCop doesn't blow chunks on those methods. In other words, if I have the following class definition:

```c#
public class Customer
{
  public void SimpleGetCustomer(string request) {}
}
```

the following test succeeds:

```c#
[Test]
public void InspectReferencedCustomerMethod()
{
  Type typeToTest = typeof(Customer);
  MethodInfo methodInfo = typeToTest.GetMethod("SimpleGetCustomer");
  Assert.IsNotNull(methodInfo, "The MethodInfo object is null.");
  Method method = Method.GetMethod(methodInfo);
  Assert.IsNotNull(method, "The Method object is null.");
}
```

However, if I change the parameter's argument type to something more complex:

```c#
public void SimpleGetCustomer(GetCustomerRequest request) {}
```

then the test blows chunks.

I spent some time digging into FxCop's assemblies via Reflector, and I was able to decompile the code underneath `Method.GetMethod()`. Everything seems fine until I hit a method called `ParameterTypesMatch()`. For some reason, that method calls `StripModifiers()` to check to see if the types of the parameters that were found from a `GetParameters()` call on the given `MethodInfo` reference matches the parameter types found on the `Method` object it got when it called `GetMembersNamed()` using the name of the method given. I know, that sounds a bit confusing, but the problem is that `ParameterTypesMatch()` blows up on non-mscorlib types when they're checked for equality, even though the two `TypeNode` references are the same types.

I'll try to contact someone on the FxCop team about this. This may not be a bug but it sure is smelling like one.

> Published: 05.20.2005 10:34:34 AM CST