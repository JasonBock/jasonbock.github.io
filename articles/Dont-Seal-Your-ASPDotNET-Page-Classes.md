---
title: Don't Seal Your ASP.NET Page Classes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Don't Seal Your ASP.NET Page Classes

As I was adding support for `<wfw:commentRss>`, I decided to seal my `Page` class that implemented my `CommentRss.aspx` page. You know, something like this:

```c#
public sealed class CommentRss : System.Web.UI.Page { 
  // ...
}
```

Oh, man, was that a bad idea! I started getting all sorts of weird parsing errors, but since my custom error page was showing all sorts of goofy tag parsing errors it took me a while before I finally set `customErrors` to `RemoteOnly` and pruned the source code before I finally saw this:

```
Compiler Error Message: CS0509: 'ASP.CommentRss_aspx' : cannot inherit from sealed class 'JasonBockDotNet.Web.Pages.CommentRss'
```

Oops! I tend to seal my classes by default, but I usually don't do much ASP.NET development, and when I do I usually leave the code-behind file alone. For some reason I thought I'd be a super-assuming genius and just mark the class as `sealed` - I mean, c'mon, how much harm could one possibly create by doing that? Apparently, quite a lot!

> Published: 01.22.2005 05:41:40 PM CST