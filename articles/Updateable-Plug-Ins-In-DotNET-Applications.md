---
title: Updateable Plug-Ins in .NET Applications
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Updateable Plug-Ins in .NET Applications

Two of my favorite .NET-based tools, Paint .NET and Reflector have plug-in frameworks so developers can extend these programs with new functionality. I've written a plug-in for Reflector, and I've noticed that there's no built-in mechanism for it to automatically download newer version of my plug-in (and any dependent assemblies). I tried to put my add-in on my web site and have Reflector try to add the assembly from that location, but no dice - it wouldn't load it [1]. Paint.NET seems a bit worse in that you're forced to put your add-in in a well-known subdirectory; there's no way (that I can tell) to specify another location for a plug-in in .NET.

This is a bit disappointing, both as an add-in and a user. As a developer, I want updates to go out to my users without them manually getting the new assembly and updating their local version. That's error-prone and cumbersome. I applaud Paint.NET and Reflector as tools, but it would be nice if they went a bit farther with their plug-ins by (somehow!) supporting automatic updates.

[1] Of course, I could be completely wrong about Reflector - maybe it can load add-ins from trusted web site locations, but I couldn't get it to work.

> Published: 03.25.2006 10:22:10 AM CST