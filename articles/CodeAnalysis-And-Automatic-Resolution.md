---
title: CodeAnalysis and Automatic Resolution
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# CodeAnalysis and Automatic Resolution

(Sorry, I can't upload pictures at the client (not easily at least) so just use your imagination ...)

If you've enabled CodeAnalysis in your VS 2005 project, you've noticed that the integration is kind of lacking. There are new features coming, but one that would be kind of cool is to allow rules to generate code to "fix" the problem. This wouldn't be applicable in all cases as some resolutions would be pretty complex, but **if** the rule wants to try and fix the problem, then let it. Maybe add a new interface like `IRuleCodeResolution`, so when I get the "CA1014 : Microsoft.Design : 'ThisAssembly' should be marked with CLSCompliantAttribute and its value should be true", I can right-click on that warning in the *Error List*, and I get the "Resolve Issue With Code", and I pick which file the new code should be generated (probably AssemblyInfo.cs), and it magically adds the `CLSCompliantAttribute` to the code base.

This is just a pie-in-the-sky idea, but for some rules the resolution is really easy, and if something could generate the code to resolve it for me, so much the better!

> Published: 01.26.2007 12:44:42 PM CST