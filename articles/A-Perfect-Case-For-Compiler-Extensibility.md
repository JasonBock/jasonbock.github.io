---
title: A Perfect Case For Compiler Extensibility
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Perfect Case For Compiler Extensibility

As I've been diving into the world of WCF, I'm using `IsOneWay` on some of my methods, like this:

```c#
[OperationContract(IsOneWay = true)]
void UseIt(Request message);
```

Now, according to the SDK:

> The IsOneWay property indicates that a method does not return any value at all, including an empty underlying response message. This type of method is useful for notifications or event-style communication. Methods of this kind cannot return a reply message so the method's declaration must return void.

So ... if I do this:

```c#
[OperationContract(IsOneWay = true)]
string GetItBack(string input);
```

The compiler doesn't raise a fuss, but if I try to call this on a service implementation, I'll get the following exception:

```
System.InvalidOperationException: Operations marked with IsOneWay=true must not declare output parameters, by-reference parameters or return values.
```

This sucks.

Why? Because there's no real mechanism in place to allow someone (either from Microsoft or a third-party or in-house developers) to **extend** the compilation process. I have to wait until I run the code to see that something is really messed up. True, you could write a custom FxCop rule that was hooked into VS's build process via CodeAnalysis (something I did before VS 2005, but it was all command-line driven), but something about that doesn't seem quite right. CodeAnalysis, to me, is all about informing the developer of things they may really want to think about, but they're not "errors". Hell, even Microsoft's .NET assemblies throw tons of FxCop warnings - they probably have legitimate reasons for doing this (though to what degree they do, I'm not sure). Having a return value on a one-way operation is clearly an error, and it really should be caught at compile-time, but since day 1 there's never been a way to make compilation extensible (not very easily at least).

There are definitely options there. I'm hoping that someday there's a well-defined ".NET way" to have this extensibiliy available so I'll know when I press Alt+B+U that I screwed something up [1].

[1] Yes, writing a unit test would expose this right away. I completely agree. But compilation would catch it even faster.

> Published: 01.22.2007 08:17:58 PM CST