---
title: When Method Overloading Goes Haywire
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When Method Overloading Goes Haywire

Let's say you have the following code:

```c#
public class ParameterOne { }

public class ParameterTwo { }

public static class OverloadedParamsIssue
{
  public static void CallOne(Guid value, params ParameterOne[] parameters) { }

  public static void CallOne(Guid value, params ParameterTwo[] parameters) { }
}
```

Now...how do you call it?

```c#
OverloadedParamsIssue.CallOne(Guid.NewGuid());
```

This gives the following error:

```
Error 1 The call is ambiguous between the following methods or properties: 'Playground.OverloadedParamsIssue.CallOne(System.Guid, params Playground.ParameterOne[])' and 'Playground.OverloadedParamsIssue.CallOne(System.Guid, params Playground.ParameterTwo[])'
```

Big deal, right? Well, this is the same issue that's currently in the `Expression` class:

```c#
public static ListInitExpression ListInit(NewExpression newExpression, params ElementInit[] initializers);
public static ListInitExpression ListInit(NewExpression newExpression, params Expression[] initializers);
```

I found out about this when I was trying out Jomo's code for visiting expression trees. If you copy the code and try to run it, you'll get an error similar to this. To fix it, I can do this:

```c#
Expression.ListInit(lin, (Expression[])null)
```

which will get the code to compile ... but it's ugly, and it won't work. I dug into Reflector to look at the method implementations and it says it will throw an error if the array in either method is null. Damn! That actually seems to violate the spirit of the `params` keyword, in that the values are **optional** if the argument is a `params` array.

On a whim, I tried this:

```c#
Expression.ListInit(lin, li.Initializers)
```

and that seems to work.

The "correct" fix IMHO is that the `Expression` class should have not had the array parameters optional, or provided another overload that just took one parameter. But if you add that extra overloaded method, then you're saying that other two methods really should not be `params`-based.

The moral of the story is, be really careful when you're overloading methods that use `params`. And if you're going to use `params`, make sure that the values are really optional!

> Published: 02.24.2009 11:51:01 AM CST