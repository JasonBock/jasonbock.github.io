---
title: My Tests Are NOT Ignored!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# My Tests Are NOT Ignored!

Damnit. I'm not a big fan of MSTest.exe and how it's integrated into VS. Frankly, even in 2008 there's still major deficiencies in the tool/framework/architecture. But by this time, you'd think they'd have the following issue fixed.

I create a `[TestClass]`. I write some tests. I change it to `[TestClass, Ignore]` because I have to (very long story). I go back and change it to `[TestClass]`. I right-click and say "run my effin tests".

Oh, it thinks all of the tests within the class are still ignored. So I have to go into the properties window and unignore **every freakin' test** in that class. And I can't highlight all of the tests and enable all of them in the property window in one shot - for some reason I can't do this.

That's terrible. I've seen this in 2005, and I'm still seeing it in 2008.

C'mon, Microsoft, **FIX THIS**.

Thank you.

> Published: 03.24.2008 10:10:57 AM CST