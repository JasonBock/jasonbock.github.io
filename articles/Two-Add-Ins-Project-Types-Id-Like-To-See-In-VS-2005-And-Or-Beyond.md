---
title: Two Add-Ins/Project-Types I'd Like to See in VS 2005 and/or Beyond ...
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Two Add-Ins/Project-Types I'd Like to See in VS 2005 and/or Beyond ...

I've had two ideas on the back-burner that I think would be cool to do in VS 2005 and above, but I know that I'll never be able to get them done (or even attempt to do it - I just don't have enough time). Maybe someone can run with this and release the bits for all to use (or maybe there's already something out there that I don't know about). Here are the ideas.

**Multiple Language Project Type**: Basically, this project type would allow you to use any code file type you'd want, so long as it's one installed on your machine (C#, VB, J#, etc.). So, you could have .cs files and .vb files and all sorts of .NET-compileable (if that's even a word) files in one project, and through the usage of that magical tool ILMerge it would combine all the individual assemblies compiled for each language and pile 'em all into one. Of course, there are definitely caveats with this approach. Classes in one C# file couldn't see classes in a VB file. Type clashing may occur during the merge. Each individual compilation would have to generate temporary assemblies that would be merged into one. But what I find interesting about this idea is that I could easily jump into a language for a specific scenario (e.g. VB works easily with Office APIs, much easier than C# does - it's usually less code) without creating another assembly.

**IL Project Types**: Wouldn't it be cool to create assemblies using IL? OK, maybe not "cool", but it would be really interesting, especially if Intellisense was involved. True, there's some tool called ILIDE# (details are here), but I'd like to do that right in the VS world, rather than having yet another editor to write code in [1]. I'm sure it could be done, and I think it would make IL coding ... tolerable if Intellisense was involved. And then you could take this project type and use it in the "Multiple Language Project Type" I mentioned before!!

Again, maybe there's ways to do these ideas already, but I haven't been able to find them.

[1] I think F# has the ability to inline IL, but I'm not sure on that. Yet another reason to get F# installed!

> Published: 06.14.2006 01:15:28 PM CST