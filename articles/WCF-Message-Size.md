---
title: WCF: Message Size
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WCF: Message Size

One of the aspects of WCF that has bitten us a number of times is message size - specifically, the maximum message size that can be received by a service. Usually we've been able to catch when a message has gone over the configured limit because the service operation was not one-way, and so we received immediate feedback from the received exception. After some research we were able to quickly fix the problem by changing the `maxReceivedMessageSize` in the binding. Yesterday, though, one of our services that uses MSMQ as its binding was failing, but we weren't getting an exception. The client would make the call successfully, and the message would get into the queue, but the service wasn't able to pull it out, and all we'd get is the `Faulted` event with no exception information. A post hints that you can get at the `$exception` local in the `Faulted` event in VS ... but I couldn't find any such local variable of that name. In any event, I was finally able to figure out that the message has changed slightly and had gone over the 65K limit, so the service couldn't read the message from the queue. We upped the configuration value and all was well again (and what was really cool is, since the queue is transactional, when we restarted the service all those messages got consumed and processed correctly - woo, hoo!).

So even though I've been able to figure out the problem, it would've been really nice to get the exception object when the service faulted. If you know of a way to do this with services that use MSMQ as its binding, please let me know - thanks!

> Published: 06.29.2007 07:06:27 AM CST