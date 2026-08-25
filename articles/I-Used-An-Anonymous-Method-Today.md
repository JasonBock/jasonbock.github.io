---
title: I Used an Anonymous Method Today!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Used an Anonymous Method Today!

OK, there are bigger things that have happened in my life. But since .NET 2.0 came on the scene, the one new feature that I didn't see using a lot of was anonymous methods. Generics - that was an easy one to exploit. But anonymous methods? I've seen 'em before in Java, and I was never impressed by them, and, more to the point, I really didn't get where they could be used.

Well, today I had to generate a Javascript on the fly via values in a `List`. The end result had to look something like this:

```js
new Array('value1', 'value2', 'value3')
```

Well, I thought of a bunch of ways to do this, but after starting at the problem for a bit, I came up with this (brace yourself!):

```c#
StringBuilder jsArray = new StringBuilder();
jsArray.Append("new Array(");

jsArray.Append(string.Join(",", elements.ConvertAll(
  new Converter<string, string>(delegate (string ID) 
  { 
    return string.Format("'{0}'", ID); 
  }
  )).ToArray()));

jsArray.Append(")");
```

Cute, huh [1]? Well, it works!

[1] I showed it to a co-worker, and he mumbled something about "job security" and "consultant coding styles". I have no idea what he's talking about.

> Published: 11.10.2005 09:15:27 PM CST