---
title: Location, Location, Location
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Location, Location, Location

While writing some code today, I wanted to create a validation-type class that would check to see if the given data fit a prescribed set of rules (or, in other words, is the damn zip code the user gave me correct?). Being in a interface-driven development mood, I was about to create an `IValidate` interface, but I decided to check out the .NET SDK to see if something like this was there. Sure enough, there's an `IValidator` interface that contains the members I need (e.g. `IsValid`, `Validate`, etc.). Whoopee! All I needed to do is reference the assembly it resides in.

Oh. It's in `System.Web.dll`.

Um ... this isn't a web project, and I'd rather not reference that DLL that contains over 700 type definitions just to get one interface definition.

I know, there's worse things to bitch about than this. Generally, .NET programming is a nice world to be in. Most of the time, the packaging with assemblies and namespaces is good. (In fact, life in general has been pretty good lately, but that's another story). But my point is: why put this interface into an assembly that's pretty much all about web-based projects? Shouldn't `IValidator` be put into `mscorlib.dll`? Or (even better) maybe in a future version of .NET, they can have a `System.Interfaces.dll` assembly that contains interface definitions that are generic and can be implemented by classes throughout the .NET world. Existing interfaces (e.g. `System.Web.UI.IValidator`) could be refactored to inherit from these new interfaces - this shouldn't break code (much).

Just a thought that hasn't been thought out much ;).

> Published: 07.10.2003 06:00:22 PM CST