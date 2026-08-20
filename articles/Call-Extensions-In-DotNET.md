---
title: Call Extensions in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Call Extensions in .NET

A couple of days ago, I starting thinking about C#, and 3.5, and extension methods, and missing methods, and ... well, I came up with an idea that isn't done yet, but I wanted to post it to see what people think.

Basically, it's the idea that you could do something like this:

```c#
int result = this.Call<int>("Add", 1, 2);
```

Where there would be an extension method defined as follows:

```c#
public static class CallExtension
{
  public static T Call<T>(this object target, string methodName, params object[] arguments)
}
```

There would another version for `Call()` where there is no return type, but you get the picture.

Of course, if the method could not be found, it would be nice to have a "missing method" hook around. One idea would be to have a delegate/event somewhere that you could hook when a method couldn't be found, or add the capabilities to register a method for a specific type, so when `Call()` can't find the method on the type, it could look for the method somewhere else. Another issue is if you're trying to call a method with `ref` or `out` parameters. Right now my `Call()` implementation (which uses `System.Reflection.Type.GetMethod()`) just gleans type argument information form the `params` array. If you're looking up a method that has `ref` or `out` arguments, you need to pass that into `GetMethod()`. Therefore, `Call()` needs to be overloaded like this:

```c#
public static class CallExtension
{
  public static T Call<T>(this object target, string methodName, Type[] argumentTypes, params object[] arguments)
}
```

And now the caller has to do something like this (assume there's another `Add()` version that has two `ref int` arguments):

```c#
Type[] argumentTypes = new Type[] { typeof(int).MakeByRef(), typeof(int).MakeByRef() };
int result = this.Call<int>("Add", argumentTypes, 1, 2);
```

Which is kind of ugly. Generally I avoid `ref` and `out` arguments if at all possible in my code, but they're valid in .NET so that would have to be supported. The other issue is if you call a method with `ref` arguments, and the target method changes that argument value, you can't get that value out of the `params` array back to the caller ... at least I can't find an easy way to do that.

At this point, I'd almost jump over to JavaScript or IronRuby or some kind of dynamic language, where this stuff is common-place. But as time permits I'm going to pursue this more. I want to see if a `DynamicMethod` compilation + caching strategy would be quicker than using the Reflection API over and over again. I also want to see what I can come up with to handle missing methods.

> Published: 07.21.2007 09:42:11 AM CST