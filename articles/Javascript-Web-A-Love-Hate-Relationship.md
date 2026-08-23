---
title: Javascript + Web: A Love/Hate Relationship
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Javascript + Web: A Love/Hate Relationship

This past week I was back in the world of web UI programming. Sometimes, I really like Javascript and the power of dynamic languages. It's **so** cool to be able to add new methods to `Object` or other "types", add new fields on the fly ... even though that can be a pain to debug, it also provides a wealth of freedom. What sucks is the UI part. There are so many damn inconsistiencies between browsers it's pathetic. I don't care who did something proprietary or who isn't following what standard - the web world sucks as a programming model. I just found a doozy this week. If you need to make a synchronous `XmlHttpRequest` call, Firefox won't call your function that you bound to `onreadystatechange` - you have to call it manually after `send()` is done. There's 2 hours of my life that I'd like back, thank you very little

I know people have created nice libraries to hide browser pain from us. But I think we're hitting the end of the road with web development. There's only so many hacks you can throw on top of other hacks before the entire thing becomes too painful to endure. I really think in the coming years the browser will be history as a way to distribute applications. At least it better be - ten years from now if the best we can do is Javascript + DOM then we really suck. We need something better ...

> Published: 02.23.2007 08:36:11 PM CST