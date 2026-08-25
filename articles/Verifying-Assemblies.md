---
title: Verifying Assemblies
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Verifying Assemblies

If you've ever emitted an assembly via the classes in `System.Reflection.Emit`, you know that all it takes is one stupid little mistake to totally mess up your emitted assembly. I know this first-hand: when I was creating my Dynamic Proxy generator for my CIL book, I made opcode after opcode foul-up. I eventually got it to work, but what I really needed is a tool to check the assembly for verification errors. There's a tool provided by the .NET Framework called peverify, but at the time I wasn't using any testing frameworks (like NUnit), and I could've invoked peverify after the assembly was saved, but it never dawned on my to do that. As I was reading my diary on writing the CIL book, I realized that it's not that hard to put a thin wrapper over peverify so you can verify your assemblies at run-time all through one method:

```c#
VerificationErrorCollection errors =
  AssemblyVerification.Verify(new FileInfo("GoodAssembly.dll"));
```

If the returned collection has errors in it, you know your assembly is bogus. If you take a look at the [code](https://github.com/JasonBock/AssemblyVerifier), you'll also see how this works with dynamic assemblies. Note that I have not tested this with the 2.0 version. I also wish there was a way to turn the metadata tokens into member names so the information that's in the `MetadataVerificationError` class would have more meaning, but I don't think there's a way to turn a token into something meaningful in C#. But I wish I would've created this a long time ago - it would've saved me hours when I created that proxy generator.

If you have any questions with the code, please let me know. Enjoy!

> Published: 09.05.2005 06:05:24 PM CST