---
title: Bewildering Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Bewildering Code

Just sit back and ponder this code for a while:

```c#
StringBuilder query = new StringBuilder();
query.Append(string.Format("//{0}[", "ArrayOf" + Result.Name));
```

So it's creating a `StringBuilder` ... but then it decides to use a `string.Format()` call instead of a couple of `Append()` calls on the `StringBuilder` ... um ... OK ... but then there's a concatenation within the `Format()` call.

The ugliness ... it hurts ... it hurts so bad!

Now, there's more code to this story (i.e. there's a good reason to use `StringBuilder`), but for all that is good in this world, why not just do it this way:

```c#
StringBuilder query = new StringBuilder();
query.Append("//ArrayOf[").Append(Result.Name);
```

Now, to be fair, I'm not sure that this would actually perform better. Tests would have to verify that. But I'm guessing that it would, and frankly it just reads better this way.

> Published: 11.01.2007 07:50:34 AM CST