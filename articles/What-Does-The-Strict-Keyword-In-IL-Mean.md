---
title: What Does the Strict Keyword in IL Mean?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# What Does the Strict Keyword in IL Mean?

I was working on some IL code generation, and I stumbled across this in ILDasm:

```
.method public newslot strict virtual instance void 
  MethodFour<T,U,V>(!!T a,
    !!U b,
    !!V c) cil managed
```

I've never seen `strict` before, and I can't find a defintion of it anyway, not in the Partition docs, not in the SDK ... I'm stumped. Internet searches haven't been helpful - the only thing I could find was [this](https://web.archive.org/web/20170528181428/http://community.bartdesmet.net/blogs/bart/archive/2006/10/02/4490.aspx). What I'm **really** looking for is a way in the Reflection API to determine if a method is "strict" (whatever that means). I wish I had comments on, but if you know, please let me know - thanks!

> Published: 12.02.2006 06:24:22 PM CST