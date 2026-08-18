---
title: "Exception Handling in .NET" - Status Update
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# "Exception Handling in .NET" - Status Update

[Recently](https://jasonbock.net/articles/I-Am-So-Tempted-To-Write-The-Exception-Handling-In-DotNET-E-Book) I blogged about how I'm so tired of seeing really bad exception handling practices in production code. The comments have been encouraging enough for me to put together a very rough outline of what I want in the book. It still needs fleshing out, but I'm about 90% ready to say "yes, I'm going to do this". I think I might call it "Investigating Exception Handling" as I already have three eBooks on my web site on Reflection, WinServices, and .NET languages (I have two others on serialization and threading but I never published them). Those three eBooks are really old and outdated ... which makes me wonder if I should spend time updating some of those as well. If I would, though, I would think very hard about charging some money for the eBooks (just to get a little bit back for the effort I'd put into them), but I'd want to keep the cost very, very low.

Oh, and I found another "nugget" in the current code base that validated why an artifact like the eBook is needed:

```c#
try
{
  // interesting code goes here...
}
catch
{
  throw;
}
```

... sigh ...

> Published: 07.24.2008 08:09:21 AM CST