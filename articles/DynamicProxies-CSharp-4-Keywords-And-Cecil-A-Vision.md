---
title: DynamicProxies, C# 4.0 Keywords, and Cecil: A Vision
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# DynamicProxies, C# 4.0 Keywords, and Cecil: A Vision

I'm going on vacation for nearly 2 weeks, starting this Wednesday, so my online time will probably be dimished for a while. Before I go, I wanted to outline some thoughts I had on a project I may pursue starting in 2009. This idea literally came to me as I was driving home from work late last week. I have no idea if this is a good idea or not - consider this post the "envisioning" portion of the project. It's all pie-in-the-sky with Visio-like drawings and whatnot, plus it may be based on incorrect evidence so don't take anything I say as "truth" (e.g. my descriptions of what's going in in 4.0) .. .but you have to start somewhere!

Consider the following C# 4.0 code:

```c#
dynamic x = new Something();
x.Configure(); 
```

If you've been following the new changes for C#, you know that `dynamic` is a new keyword. Essentially, what this does is it allows a C# developer to use members on an object dynamically. We have no idea if there is a method called `Configure()` available on `x` at compile-time; we're hoping that this will work at run-time. In fact, VS 2010 doesn't give you any Intellisense when you hit the period, because it can't. It has no idea what you could invoke on `x`. Note that the work that C# does underneath the scenes to get this to work is rather messy. Basically it creates a new `CallSite` class, provides it with a bunch of information, and then invokes the target. Feel free to write a small program in C# 4.0 and view it in ildasm if you're that curious to see this in action yourself.

Here's the thing. If your object implements `IDynamicObject`, then you can get notifications whenever the client wants to invoke [1]. This includes everything, even non-virtual calls.

This is of interest to me because I wrote my [DynamicProxies](https://github.com/JasonBock/DynamicProxies) framework years ago. And I've never been able to catch non-virtual calls. Now I can!

Well, there's a catch. Let's say I wrote my code like this:

```c#
// assume the "this" implement IInvocationHandler
dynamic x = new Something().CreateProxy(this);
x.Configure();
```

Assuming that I added the feature in DynamicProxies to implement `IDynamicObject` on the proxy (and of course giving the client the ability to enable or disable this feature whenever they wanted to), this would work. That is, if `Configure()` was non-virtual, I could now get the call. But this requires the user to switch from "dynamic" to "var" at the right time.

Enter MSBuild and [Cecil](https://github.com/jbevain/cecil).

Here's the idea. I create a post-build framework based on Cecil called Cecil.MSBuild (or something like that). It's a generic framework that allows developers to plug in various Cecil-based modifiers on the compilation result of C# (or VB or F# or any managed compiler). In this case, I'd create a plug-in that would look for any "CreateProxy" call, and replace subsequent invocations to that call as a "late-bound" dynamic call if the method is non-virtual, and keep it the same if it is virtual.

My guess is that this wouldn't be easy to pull off, but again, I'm just envisioning at this point :).

So my idea is this:

* Create a new MSBuild-based framework using Cecil that would allow anyone the ability to modify an assembly post-build. Not too hard, just makes it a bit easier for .NET devs to add this modification business into a pre-existing MSBuild fle (which the .csproj and .vbproj files are).
* Update DynamicProxies to implement `IDynamicObject` (based on a configuration value). All non-virtual calls would be handled by this interface; virtual calls would be deferred to the proxy implementation.
* Create a post-build process that would change non-virtual calls to proxy-based objects to be "semi-dynamic". Virtual calls would still be done directly to the proxy. Non-virtual calls would be done against the `MetaObject` provided by the proxy's implementation of `IDynamicObject`. This would eliminate the need for developers to declare their variables as dynamic.

Let me know what you think.

[1] See [this article](https://learn.microsoft.com/en-us/archive/blogs/cburrows/c-dynamic-part-ii) for details.

> Published: 12.21.2008 02:42:51 PM CST