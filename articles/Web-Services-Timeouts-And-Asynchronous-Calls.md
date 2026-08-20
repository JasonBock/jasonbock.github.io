---
title: Web Services, Timeouts, and Asynchronous Calls
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Web Services, Timeouts, and Asynchronous Calls

I'm currently using `MSNSearchService` at the current client (although this story has to do with web services in .NET in general). We did the standard wsdl.exe code gen stuff, and most of the time we haven't run into any huge issues (well ... some issues, but I'll leave that discussion under the covers). Anyway, when I wrote my code against the service, I noticed that it had a `Timeout` property. There wasn't any documentation for it on the web site, but I assumed (yeah, that's a bad thing - you'll see why in a moment) that if I set this it would return in that time if MSN wasn't fast enough. All of our code is asynchronous (another key to the puzzle), so we wanted to make sure that we didn't hang out forever if bad things were happening on the other end.

Long story short ... we noticed that some queries were taking far longer than our configured value. I started digging into what was going on with the basics: is our configuration-reading-property-setting code working correctly? Yep. If I set the timeout value to something ridiculously short, like 1 ms, does it still work? Yep. Hmmmm ... that didn't sit well with me. So I started looking at the class definition, and I noticed that `MSNSearchService` (indirectly) descends from `WebClientProtocol`. I looked up the docs on the `Timeout` property, and guess what I found (emphasis mine):

> Indicates the time an XML Web service client waits for a synchronous XML Web service request to complete (in milliseconds).

Ugh! It's only applicable for synchronous calls, not asynchronous calls. The reason why is `WebClientProtocol` delegates to `WebRequest` to do the dirty work (at least that's what I found when I busted these types open in Reflector). The docs for the `Timeout` property on `WebRequest` are more explicit.

> The `Timeout` property affects only synchronous requests made with the `GetResponse` method. To time out asynchronous requests, use the `Abort` method.

Like I said, I should not have assumed what `Timeout` would do on my proxied web service class. But it seemed like a safe assumption at the time!

We have a workaround being created as I type - it ends up not being that bad to get the behavior we were looking for. But, once again, I've learned not to assume what an API does, even if the properties and methods seem clear enough.

> Published: 07.17.2007 08:39:36 PM CST