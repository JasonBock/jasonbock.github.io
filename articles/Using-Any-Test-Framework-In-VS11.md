---
title: Using Any Test Framework in VS11
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using Any Test Framework in VS11

**UPDATE**: I figured out how to get code coverage working. It's the "Analyze Code Coverage" button in the Unit Test Explorer. I have to find the keystrokes to use this though as going through a UI to run tests is not fun.

For a while, I used MSTest.

It's not that I liked it. It's not that I didn't like it. And just because it was integrated with Visual Studio didn't make it an automatic choice either. But as a testing framework, it was ... OK. I know others that disagree with my view, and that's OK. It doesn't mean I didn't like [NUnit](https://nunit.org) or [xUnit.net](https://xunit.net/); quite the opposite, actually. But it was there when I installed Visual Studio, and frankly, for some of the clients I've been at, that's a win to get them to use it for testing. I'd rather have my team write tests than get hung up on using "yet another framework" and spending time over which one we should pick. (I know, that's kind of sad, but nothing's perfect in life).

With VS11, you'll be able to plug in any testing framework you like. I tried that yesterday with xUnit.net and I was pleasantly surprised at how easy it was to get it up and running. The steps you need to take are simple:

* Nuget xUnit.net
* Install the prototype unit test plugin
* Write tests
* Execute tests

Simple. The same story either exists now for the other .NET-based testing frameworks or will in the very near future. I couldn't get code coverage working with my small sandbox solution, but I didn't look that hard either with it. I may be missing something obvious with that. I'm assuming that the unit test running will also produce code coverage results with the other frameworks (at least I hope it will).

This is a long time coming. At this point, I'm going to convert all my tests for my personal projects away from MSTest. I didn't hate it, but now I really don't see any need to use that framework. Other testing frameworks have some appealing options to them and with VS integration it's much more appealing to use those frameworks.

> Published: 09.22.2011 11:36:06 AM CST