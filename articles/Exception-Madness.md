---
title: Exception Madness
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Exception Madness

Recently I blogged about exception handling. Today I saw code that looked like this:

```vb
Try
  '  Lots of data access code goes here...
Catch exp As Exception
  Throw
End Try
```

And if that wasn't bad enough, someone at Magenic informed me of a "documented standard" he had to conform to:

```vb
Try
  '  ALL code within the method goes here
Catch exp As Exception
  Throw New Exception(exp.Message)
End Try
```

... sigh ...

> Published: 07.19.2006 12:25:51 PM CST