---
title: Adding Tasks for Dynamic Proxies 3.0
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Adding Tasks for Dynamic Proxies 3.0

After having an IM conversation about dynamic proxies with Aaron (which started with this post), I decided to add some features to my library that were I've been mulling over in my head now for quite some time. Frankly, they shouldn't be a surprise - if anything, you should be surprised that my library doesn't have these features yet. But hey, I gotta keep myself busy.

One of the more "controversial" features I'm thinking of adding (and this will be the easiest to go on the chopping block if it's too much work) is to add a hooking mechanism based on lambda expressions. Right now, the proxy generation requires that the caller provide at least one object that implements `IInvocationHandler` to receive the before/after calls. Unfortunately, this is an "all" mechanism - that is, I hook everything I can. One possibility is to give the user a mechanism to limit this. That is, for the following class [1]:

```c#
public class Hooked
{
  public virtual void MethodOne() {}
  public virtual void MethodTwo() {}
  public virtual void MethodThree() {}
}
```

I could create a proxy like this:

```c#
Hooked hooked = Proxy.Create(new Hooked(), 
  new HookMappingCollection(
    new HookMapping(typeof(Hooked).GetMethod("MethodOne"), true, true),
    new HookMapping(typeof(Hooked).GetMethod("MethodTwo"), true, false)));
```

Now I'd only get the before/after calls for `MethodOne()`, and only a before call for `MethodTwo()`. `MethodThree()` would not be hooked.

The other idea was to let the caller provides delegates/lambdas to methods that matches the arguments of the target method to be hooked, plus some "extra" information that the before/after needs. Frankly, I'm too tired to think of how this might work, but this would provide a lean-and-mean way of hooking. `IInvocationHandler` feels very "late-bound" and having something more type-safe would be nicer.

Anyway, that is all for now. I have no idea when I'll get all of these features in, but I like revisiting this library now and then.

[1] Note that all of the code in this post is pseudo-code and probably has compilation errors :). I'm just trying to get ideas across.

> Published: 07.01.2008 10:14:29 PM CST