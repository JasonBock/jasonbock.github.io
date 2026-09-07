---
title: Damnit! FxCop 1.21 Broke my Code!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Damnit! FxCop 1.21 Broke my Code!

In Chapter 5 of my upcoming book, I use the FxCop SDK assembly extensively to handle method parsing to track down exceptions. There was a static method on the Method class called `GetMethod()` in the 1.091 version that returned a `Method` object based on an assembly name, a module name, and a method token. After I wrote the chapter, a new version of FxCop came out: 1.21. I figured that I'd wait to upgrade my code to use the new SDK assembly after Chapter 5 was reviewed. Well, I got the review of Chapter 5 yesterday, and, unfortunately, the transistion wasn't very smooth. Basically, a method that was marked as `public` in 1.091 was changed to `assembly` in 1.21. This ended up breaking my code as I could no longer access that method. Furthermore, I couldn't find anything in the SDK that replaced this functionality either.

Granted, this assembly is probably not being used by a lot of people (the fact that it's not documented limits its usage as well). But when a method is `public`, you run the risk of breaking somebody's code when you change that accessibility level to `assembly` or `private`. Overall, I like the functionality of this SDK assembly, but I wish the designers would document it, especially what got changed from version to version (unless this information is there and I'm just missing it in the installation directory).

All in all, the fix wasn't too hard. I ended up using some Reflection-based code to invoke the method, and all was back to normal. But now I feel like my code has a hack in it because I'm invoking a non-public method.

> Published: 08.06.2003 08:29:33 AM CST