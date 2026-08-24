---
title: On-The-Fly Assembly Verification
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# On-The-Fly Assembly Verification

When I was writing my CIL book, I was in the `System.Reflection.Emit` pretty heavily, and it was a love-hate relationship. The love part was due to being able to do anything that was possible in the CLR (well, pretty much anything), and the hate part was that it was really tricky to get right. At the time, even though I knew about the peverify tool, it never dawned on me to use that to make sure my dynamic proxy code was doing things correctly (I also needed unit tests ... hmmmm ... maybe I should revisit that proxy project again ... ). The real issue is that peverify is only accessible as a console application (an unmanaged one at that!); there's no API in .NET to verify an assembly (at least none that I'm aware of). So what I've done is write a thin wrapper around peverify to make dynamic assembly verification easy. Basically, it works like this:

```c#
AssemblyBuilder goodAssembly = this.BuildAssembly();
VerificationErrorCollection errors =
  AssemblyVerification.Verify(goodAssembly);
Assert.IsNotNull(errors, "The collection is null.");
Assert.AreEqual(0, errors.Count, "The count is incorrect.");
```

Of course, if there are IL or metadata errors, they will show up in the collection.

If you're doing dynamic code generation from the Emitter, you owe it to yourself to use something like it. It's so easy to mess up a dynamic assembly, at least I think it's far easier to create bad assemblies via the Emitter classes than it is to write C#. Knowing right away that you really screwed up helps tremendously during the testing phase. Maybe someday assembly verification will be accessible in the .NET framework, but until that time feel free to use my code. Enjoy!

> Published: 06.14.2006 01:03:01 PM CST