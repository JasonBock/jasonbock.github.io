---
title: Another CodeDOM Question: Debug Files
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Another CodeDOM Question: Debug Files

Well, I solved most of my issues that I had yesterday with the CodeDOM classes (although I'm sure I'll have more in the next day or two as I start to really dive into the code modification portion of my idea), but I just saw one today that's left me confused. When a command-line switch comes throught to create a debug build (/debug) I set the `IncludeDebugInformation` property to `true`. If the resultant assembly is a DLL, I get the pdb file in the directory. However, when I build an EXE, the pdb file is nowhere to be found. I found out that the EXE file is also put into a temp directory, but the pdb file isn't there.

Why is this? Am I missing something? Where does the debug file go?

> Published: 04.17.2003 03:38:43 PM CST