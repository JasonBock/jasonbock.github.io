---
title: Extending .NET Compilers
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Extending .NET Compilers

When I wrote Chapter 5 for "Applied .NET Attributes", I quickly got frustrated because you really couldn't write attributes that were applicable at compile-time (think `CLSCompilantAttribute`). Ever since then, I've been thinking about writing a .NET compiler that could be used for any .NET language and would allow you to add your own compilation rules (some of which would use custom attributes!). I finally got around to working on it a couple of weeks ago. The tricky part was going to be the custom compilation rules engine, but I realized that the FxCop SDK pretty much does exactly what I need.

The results of what I did can be found here (2026: sorry, zip file is gone). I'm calling it the *Extensible Compiler for .NET*. All of the source code is included in the ZIP file along with a Word doc that documents (to some degree) the essentials of the code. I'm considering it a 0.1 beta release, so if you find problems/issues with it please read the doc to find out how to submit bug reports and feature requests. If a lot of people end up using this and there's a significant need to broaden the code base support beyond my availability, I'll try to move the code somewhere else. Right now, I'll just keep it in a ZIP file.

Enjoy!

> Published: 03.18.2005 10:39:02 AM CST