---
title: Misuse of Implicit Conversion
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Misuse of Implicit Conversion

A while ago I posted about implicit conversions. One of the comments correctly pointed out that I was misusing implicits in the case where a conversion would cause an exception. That should be an explicit conversion. But if I did that:

```c#
public static explicit operator Percentage(decimal value)
{
  return new Percentage(value);
}
```
Then my code looks ugly again:

```c#
Program.Report((Percentage)10m, (Percentage)20m);
```

In fact it actually looks worse that the code does without any conversions, implicit or explicit (in my opinion):

```c#
Program.Report(new Percentage(10m), 
  new Percentage(20m));
```

I'm not saying implicit and explicit conversions are wrong, and I agree that I was misusing them. Sometimes you can use tricks that are too cute for their own good :)

> Published: 06.09.2008 09:55:43 AM CST