---
title: Generating Code Files From Reflector
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Generating Code Files From Reflector

I just finished a fun little side project: creating an add-in for Reflector that generates code files. Click here to get the source code - it also contains the add-in DLL in the bin\Release folder. Note that this is a VS .NET 2003 solution (so it's 1.1-based) and I'm targeting the 3.2.5.0 version of Reflector. Basically, once you get the add-in up and running into Reflector, you can select a module or a type in the main tree view window, and press Ctrl + G (or go to "Tools -> Generate File(s)..." - whatever you prefer). You'll be asked to provide a directory where all the files will go. Once you pick a directory and press "Generate Files", the add-in will create a file for every type in the module (or just the type that you selected). Here's what the tool looks like when it's in Reflector:

![Reflector Code Files](https://jasonbock.net/images/Reflector-Code-Files.png "Reflector Code Files")

Let me know if you run into any bugs. There are a couple of formatting issues that the code generators within Reflector have (e.g. the attribute syntax for VB .NET code is wrong), but there's not much I can do about that right now. However, the generated code should be close to the "real thing" - it's fun to point the add-in at mscorlib.dll and see what you get.

Finally, I did this for educational purposes **only**. I have no intention of decompiling mscorlib.dll and recompiling it for any reason (profit, power, etc.). Nor could I - the source code that's generated wouldn't compile as-is. In other words, please use it for the same purposes ;).

Enjoy!

> Published: 07.09.2003 12:27:58 PM CST