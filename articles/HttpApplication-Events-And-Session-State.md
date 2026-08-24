---
title: HttpApplication, Events, and Session State
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# HttpApplication, Events, and Session State

I'm finishing up a short gig writing a `HttpModule` to handle authentication. All was well with my unit tests until we tried to run the module on one of their dev boxes yesterday. I've written a couple of `HttpModule`s in the past, and I've always hooked the `BeginRequest` event to catch when a resource has been requested. Unfortunately, I made a bad assumption with this module, because I assumed session state would be available when this event is raised. I was wrong. You have to wait until `AcquireRequestState` is raised, which happens after `BeginRequest` (check the SDK for an ordered list of `HttpApplication`'s events).

Just goes to show you that development testing is always a good thing, reading the SDK can't hurt, and making assumptions is just stupid.

> Published: 06.09.2006 07:28:22 AM CST