---
title: Negative Null is a Nullable Int
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Negative Null is a Nullable Int

I was coding along today and I meant to type in this:

```c#
if (this.Update() != null)
```

But I actually typed in this:

```c#
if (this.Update() != -null)
```

Notice the minus sign before the `null` keyword. When I compiled this, I got a very interesting error message:

```
Operator '!=' cannot be applied to operands of type 'MyItem' and 'int?'
```

Huh. I didn't know that a negative null was actually a nullable integer!

> Published: 12.12.2006 12:31:35 PM CST