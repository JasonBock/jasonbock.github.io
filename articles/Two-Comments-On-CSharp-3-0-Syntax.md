---
title: Two Comments on C# 3.0 Syntax
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Two Comments on C# 3.0 Syntax

Eric [blogs about](https://web.archive.org/web/20140428024249/http://blogs.msdn.com/b/ericgu/archive/2005/09/14/466510.aspx) two new aspects of C#: type inference and extension methods. I like both concepts, but I have an issue with the first one. See, to use type inference, you have to do this:

```c#
var i = 3;
```

That feels so ECMAScript-like. I know some people who looked at that and said, "dear god, is C# going typeless?", which isn't the case, but the keyword tripped them up. My personal feeling is I don't like the `var` keyword. I wish I could just do this:

```c#
i = 3;
```

and forget about `var`. It's less physical typing and the meaning would be the same. I like type inference; I just don't like the keyword approach that they picked. VB 9.0 doesn't add a keyword - they make it far easier:

```vb
Dim Population = 31719
```

Note that it's still typed as an `Int32`!

I really like extension functions, though. Part of me thinks that this will aid a lot with control designers. Think `Localizable`. It's not a property on the `Control` class, but it's added in the Designer through an attribute on the class itself. It would be possible to create that `Localizable` property though the extension:

```c#
public static class LocalizableExtension
{
  public static bool Localizable(this Control value)
  {
    get { //... }
    set { //... }
  }
}
```

Actually, that code isn't quite correct. The 3.0 specification states that extensions only apply to functions. It's being considered for properties, events, and operators (see Section 26.2) - I really hope they do this! That would make extension truly kick-ass.

In the end, 3.0 features are far away, and there will probably be a fair amount of changes between now and its RTM. Hopefully they do some refinements, but I'm pretty happy with what I've been reading up on.

> Published: 09.14.2005 07:58:46 PM CST