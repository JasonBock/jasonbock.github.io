---
title: The Lack of a CodeDom Parser
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Lack of a CodeDom Parser

Well, here it is, 2005, and we **still** don't have a frackin' CodeDOM parser! 1.0 and 1.1 didn't have one, and the same story exists for 2.0. It's even more explicit in 2.0 - you get a `NotImplementedException` if you try to get a parser for C# instead of a null reference.

This sucks. Period.

Why? Because we can only do so much that makes the CodeDOM so cool. We can generate an assembly from a tree (point B to point C). We can generate code from a tree (point B to point A). But we can't generate a tree from code (point A to point B). OK, we can't make a tree from an assembly (point C to point B). But, the thing is, the CodeDOM API explicitly supports the three steps (`CreateCompiler()`, `CreateGenerator()`, and `CreateParser()`, respectively), yet the third one has never worked.

Yes, I've read this post. I hear the points. But, frankly, why Microsoft doesn't supply a CodeDOM parser doesn't make a lot of sense to me, even with these arguments. And I know someone could also say, "Don't be part of the problem, be part of the solution and write your own C# parser!" True, but I bet Microsoft has a lot more of the moving parts here and there to get this to work a lot better than what I could. This site hints that they're working on a C# CodeDOM parser, but there isn't much there.

If there was a parser, this would make some of my Extensible Compiler for .NET work even easier. I know other cases where generated source code could be modified and tweaked in the CodeDOM. Since it seems like we'll never get a CodeDOM parser from Microsoft, I tend to agree with Ted:

> This is an open call to the .NET CodeDom team: If you're not going to provide even a single implementation, then just rip the damn thing out and quit teasing me. Actually, go one step better, and actually provide what would be an awesome enhancement to .NET 2.0. Disgusting.

> Published: 11.20.2005 05:34:15 PM CST