---
title: Constructors Returning Null (?)
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Constructors Returning Null (?)

A couple of days ago I saw some code that left me speechless:

```c#
FileInfo file = new FileInfo("somefilepath");

if(file == null)
{
  // ...
}
```

Look at the code in bold. If a constructor fails, you get an exception - I've never seen the new operator return `null`.

Is this just wrong as wrong can be, or am I missing something?

> Published: 09.19.2007 11:48:34 AM CST