---
title: Problems With Eiffel for .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Problems With Eiffel for .NET

Well, I spent a couple of hours with Eiffel for .NET last night, and it drove me a little bonkers. I tried to create a simple DLL-type assembly, but the blank project kept insisting on having a root class with the default make procedure, which, in a DLL, you don't need. Entry points are only useful in EXEs, so there's no reason to specify a method that is the entry point (this is assuming I understand what they mean by the default make procedure of a project). Plus, it wouldn't compile any other class definitions I would make in the project, and it would completely ignore {NONE} on my features, which I thought would turn to feature to `private`, but it didn't. Also, there's no Intellisense, at least not as far as I can tell. Plus, the compilation is god-awful slow. Oh, and trying to rename a .e file doesn't make the Eiffel plug-in very happy.

Eventually, I got too tired to think so I went to bed. I'll try to pick up my investigations this weekend. But it's frustrating when I can't even get the simple stuff done, like create an assembly and reference it in a C# project. I know it can be done, but I looked at a bunch of the example Eiffel projects and none of the ones I found were of the DLL type. Hell, even the Python for .NET compiler makes DLLs that I can use in C#, and that's an experimental compiler! I'm sure there's a way to do this, and I wasn't thinking straight last night. But it definitely wasn't intuitive, that's for sure.

> Published: 11.01.2002 10:00:00 AM CST