---
title: Cool Use of Attributes in NCover
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Cool Use of Attributes in NCover

I decided to visit NCover's site today. Seems like it's getting some life again, which is a good thing - I've always like NCover. I stumbled across this post and I think it's an excellent way to use attributes without introducing a dependency to an assembly you probably don't want. Basically, you can specify methods in your code that you don't want NCover to worry about when it's figuring out how much of your code is covered. It does this by providing a list of attributes that, if found on a class or method, will be ignored by NCover. But, NCover doesn't require you to mark your code with an attribute it has in its framework assembly. Therefore, if you have a test method like this:

```c#
[NUnit.Framework.Test]
public void TestButIgnoreInNCover() { }
```

You can tell NCover to ignore all NUnit test methods without having to "re-mark" the method like this:

```c#
[NUnit.Framework.Test, NCover.Framework.Ignore]
public void TestButIgnoreInNCover() { }
```

This is really sweet, especially since you probably don't want (or need) to add another assembly dependency in your code base, and the code you do want to ignore is (probably) already marked with an consistent attribute (like `NUnit.Framework.Test`). Very nice!

> Published: 10.09.2006 09:58:24 AM CST