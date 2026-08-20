---
title: Understanding Tail Calls
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Understanding Tail Calls

[This](https://learn.microsoft.com/en-us/archive/blogs/davbr/enter-leave-tailcall-hooks-part-1-the-basics) is an excellent write-up of tail calls in .NET. There was one interesting tidbit at the end:

> Since tail calls can be elusive to find in practice, it's well worth your while to use ildasm/ilasm to manufacture explicit tail calls so you can step through your Tailcall hook and test its logic.

I think it's time to revisit my extensible compiler and make it a phased compiler, so you can add in your own phases at will. Right now that compiler just has a 2nd phase that's hard-wired to use FxCop-based rules to create new compilation errors. That's too limiting; adding in a phase that uses Cecil to do IL searching and rewriting on a gen'd assembly from csc.exe would be nice. Then I could look for code like this:

```c#
public void DoIt()
{
  DoThat();
}

private static void DoThat() {}
```

and add `tail.` "fix-ups" to the `DoThat()` call. I could also get fancy and do something like this:

```c#
[assembly: AddTailCalls]

public void DoIt()
{
  DoThat();
}

private static void DoThat() {}
```

This could also be a compilation switch as well - i.e. add tail calls where you can, or not at all. These are just off-the-cuff ideas, but, again, having the ability to extend the compilation process allows you to do things that the "normal" language and associated compiler won't allow or do.

> Published: 07.03.2007 12:26:23 PM CST