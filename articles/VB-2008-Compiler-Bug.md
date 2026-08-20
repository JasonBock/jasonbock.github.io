---
title: VB 2008 Compiler Bug
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# VB 2008 Compiler Bug

Read about it [here](https://web.archive.org/web/20081204231112/http://msmvps.com/blogs/kathleen/archive/2007/11/19/how-to-sidestep-a-visual-basic-compiler-bug-in-rtm.aspx) (there's more info [here](https://web.archive.org/web/20150326045237/http://msmvps.com/blogs/kathleen/archive/2007/11/19/more-on-the-vb-9-0-compiler-bug.aspx)). Personally, I don't see this as a big deal. You have to encounter a specific set of circumstances that probably won't occur a lot at all (if I understand the post correctly). You have to have a constrained generic in an assembly that's referenced both as a file reference and a project reference (something I personally never do in a solution), and you have to have a `Try` block (I'm not sure what she means by this - does she call code in the `Try` block that uses the constrained generic?). It definitely seems like it's a bug that Microsoft is aware of and will try to resolve in the future. But again, the circumstances seem like an edge case, and the solution to sidestep the bug seems easy.

If it was a compiler bug that never declared locals that were declares as `Integer`s, that would be a big issue (one that probably would have a snowball's chance in hell of making it to RTM). Hopefully this won't lead to FUD-like reporting on the Internet that VB sucks, Microsoft sucks ... oh wait, that happens anyway 8-).

> Published: 11.20.2007 08:45:19 PM CST