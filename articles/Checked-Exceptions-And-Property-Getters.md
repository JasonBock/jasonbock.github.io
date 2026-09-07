---
title: Checked Exceptions and Property Getters
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Checked Exceptions and Property Getters

OK, I know Java has them, and .NET doesn't. I'm not going to argue what's better or what's worse - the die has been cast, so to speak (although maybe .NET via attributes could do the same thing ... never mind). Anyway, I just got an unexpected exception when I try to check to see if the `UrlReferrer` property is null or not. Here's the code:

```c#
if(this.Request.UrlReferrer != null &&
  this.Request.UrlReferrer.ToString() != null)
{
  // ...
}
```

My exception log shows that when `get_UrlReferrer()` is invoked, a `Uri` object is constructed. A quick visit to Anakrino confirms this as the `_referrer` field isn't created until it's needed by a client. However, the `try ... catch` block only tries to catch `HttpException` types; I'm getting a `UriFormatException`, which basically means that the property throws an exception. It could come from `Uri`'s constructor, or it could also be coming from a call to `GetKnownRequestHeader()` on a `HttpWorkerRequest` object. This call is getting a value of 36, which is the `HeaderReferer` value in `HttpWorkerRequest`.

Yikes.

I thought that my code was being defensive in that it was checking to see if the property was not null before I tried to use it. But the getter actually throws an exception just by accessing it! I don't know - this feels wrong at first glance, but maybe there's a just reason why it was implemented this way. But my gut is telling me that you shouldn't get penalized just for accessing a property's value. Usually, .NET's documentation will tell you when something will throw an exception, but it doesn't with `UrlReferrer`.

It's not that I can't get around this problem. Just set a temporary `Uri` value equal to the result of the property getter and wrap that around a `try ... catch` block. It's not pretty, but it'll work. It's just that getting this exception was truly unexpected. I've never seen where getting a property's value actually caused an exception. And the funny thing is that the getter in this case has exception handling that will set the internal field's value to null if something unexpected happens in `Uri`'s constructor. I got the feeling they need to add a little bit more to the exception block (but I could be way off-base with my analysis).

> Published: 04.05.2003 10:05:13 PM CST