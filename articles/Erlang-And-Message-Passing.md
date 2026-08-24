---
title: Erlang and Message Passing
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Erlang and Message Passing

I've never programmed in Erlang, but reading this article on Erlang and asynchronous programming made me smile:

> Let's redesign Java with message passing concurrency in mind. Throughout the article we'll use familiar syntax and principles. If you're interesting in learning how Erlang actually implements these principles you'll find plenty of resources with your favorite search engine (let's pretend for the moment that there is more than one good search engine). Erlang is to locks what Java is to pointers. Erlang designers realized that Dijkstra style locks are a low level mechanism that doesn't belong in a high level language. Locks were the first to go. So let's remove all references to locks from Java. The synchronize keyword goes. So do notify/wait functions. All we're left with are pure threads.

Replace "Java" with ".NET" for the CLR folks out there if you'd like - the quote will mean the same thing. I'm not suggesting that Erlang is an amazing language because of the ideas it conveys, but what resonated with me is the idea of isolating the threads and using messages for thread communication. This is pretty much what I did at a recent client for our asynchronous methods. We created a framework for asynchronous methods that took a message in before it executed and returned a message when it completed. This isn't the same thing that Erlang is doing, but what it did was isolate the thread processing into discrete units that could be composed by other methods. It was clean (at least in my mind it was clean!) and it made things pretty easy to manage.

> Published: 08.14.2006 09:20:58 PM CST