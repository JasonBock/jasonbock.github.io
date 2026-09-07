---
title: Twisting My Head Around `IsNothing`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Twisting My Head Around `IsNothing`

`IsNothing()` in VB .NET is a fun method. It's better than using `If someVariable Is Nothing`, but both accomplish the same goal and it's pretty up to personal taste in terms of what you use (although I always change the code in our project that doesn't have one exit point, then another guy changes it back ... it's a fun religious war ;) ). Today I ran across some code that made my brain do some odd contortions. You may find it obvious, but it looks pretty funny at first glance:

```vb
Dim booleanValue As Boolean = False
Console.WriteLine("IsNothing(booleanValue) = Nothing? {0}", 
  _IsNothing(booleanValue) = Nothing)
```

OK, guess what the output is going to be. Ready?

```
IsNothing(booleanValue) = Nothing? True
```

Uh ... huh. OK, even though a `boolean` is a value type, it's still a valid reference, right? So `IsNothing()` should return `false`, correct? Well, that's true, but comparing `false` to `Nothing` returns `true`?

Here's some more code to help clarify the issue.

```vb
Console.WriteLine("True = Nothing? {0}", True = Nothing)
Console.WriteLine("False = Nothing? {0}", False = Nothing)
```

With the output being ...

```
True = Nothing? False
False = Nothing? True
```

For some reason, comparing `true` to `Nothing` is `false`, and vice-versa. I guess it makes some sense in a weird way, but in my mind neither one should return true as they're both valid references.

Of course, this was found during a hairy debugging session today, and it wasn't the cause of the problem, but when I found the code (well, it's not the exact same code but the idea I'm showing is close to what was in our code base) I burst out laughing. It's just bizarre! And it's late, so I'm not going to spend any more time thinking about these results. Just one of those weird things you find in code (one guy recently showed me a trick using a `Select Case True` construct. It felt dirty, but it worked ;) ).

> Published: 05.15.2004 01:21:36 AM CST