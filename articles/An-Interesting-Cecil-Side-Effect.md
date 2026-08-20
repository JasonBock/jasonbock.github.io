---
title: An Interesting Cecil Side-Effect
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# An Interesting Cecil Side-Effect

I've been playing around with [Cecil](https://github.com/jbevain/cecil) here and there for the past couple of months. It's a very nice library to do all sorts of fun assembly trickeries. One thing I've noticed when I use it on an assembly that I'm adding code to is the resulting assembly is smaller. This always seems to be the case - I've run a simple piece of code like this:

```c#
AssemblyFactory.SaveAssembly(
  AssemblyFactory.GetAssembly("MyAssembly.dll"), "MyAssembly.dll");
```

and the assembly is reduced in size.

So what's going on? My first thing to try was to produce an IL file from ILDasm on my Quixo3D.Framework.dll assembly when it was compiled by VS 2005 in release mode (which produces an assembly 44 KB in size). The resulting .il file is 355 KB in size. Now I run my Cecil minimizer code on the assembly, and it's reduced to 32 KB in size. I run ILDasm on it, and the resulting file is only 350 KB in size. I tried to run a diff on the files but Cecil must do some reordering of the assembly's contents because the order of its contents is much different after Cecil saves the assembly, so it's really hard to figure out what's going on. So I tried another approach. I used my FileGenerator add-in for Reflector to generate the source files in IL and compare them that way. But that was problematic because Reflector wouldn't produce IL files for two of my classes ... which seemed really odd but it is what it is. I tried using a diff add-in, but it didn't help - it couldn't tell the difference between the two assemblies. Well, not off the bat - I had to recompile my assembly in VS 2005 with a different version number and that start showing differences. However, the add-in's UI is pretty painful to use - it doesn't scroll so I can't see what's different easily. Exporting the results generates a 600 KB XML file...whee, just what I wanted (not!). I can see that Cecil generates larger `.maxstack` values than what .NET generates, but other than that the code looks the same. I'm guessing it has to do something with the assembly metadata but I just can't tell.

I want to do some more investigation, like running complex applications after the assemblies have been minimized and see if things still work as expected. I really want to figure out why the files are getting smaller, but I'm just too tired - I'll spend more time on this later.

UPDATE (11.20.2007 - 8:50 AM): I tried this minimizing technique on the assemblies for my client's web application. Before I ran the minimizer, the total size of the 12 DLLs was 1,077,248 bytes. After minimization, the size was 951,808 bytes. This is about a 12% reduction in size, and I didn't notice any breaks in the functionality of the application. This is definitely **not** a recommendation that you start running your assemblies through Cecil just to reduce the size of the assemblies (I have no idea if the perfomance changes in any way), but it's still an interesting effect!

> Published: 11.19.2007 09:46:49 PM CST