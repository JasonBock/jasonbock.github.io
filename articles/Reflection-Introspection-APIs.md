---
title: Reflection/Introspection APIs
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reflection/Introspection APIs

During my time as a .NET developer, I've ran into a number of APIs that allow you to tear into an assembly and its constituent parts to see what's going underneath the hood. The most common one is `System.Reflection`, which allows you find out all sorts of information about an assembly. It also allows for dynamic invocation of methods. However, Reflection (before 2.0) didn't give you the ability to read the IL of a method. Now there's `GetMethodBody()`, but that just gives you the bytes; you're pretty much on your own to figure out the opcodes.

So, the Reflection API is pretty much a dynamic view of your code in that you can run methods, properties, etc., on code. It's slower than just directly invoking the method on an object or class, but for plug-in architectures it's pretty sweet. There's also the Introspection API, which you can find in the FxCop drop. I think the `Microsoft.Cci` assembly is really a part of the Phoenix libraries [1], but I'm not sure [2]. In any event, the Introspection API doesn't allow for invocations like Reflection does, but it has a really nice API over the method instructions. I use it for my extensible compiler - it's pretty easy to see what's going on underneath the scenes of a method without a lot of pain. The problem is that the only way to get this API is to install FxCop. That's not a big deal, but if the Introspection API was rolled into the BCL (like `System.Introspection`) that would be really cool. Maybe it is there in 2.0, but I haven't found it yet.

In a sense, Reflector also has an API to dig into assemblies, but it's not as full-blown as the Introspection API. Also, Reflector's API is meant to be used by plug-ins; it seems like its intention is not to be used by other assemblies for the purpose of Introspection. Reflector's API is good at getting the contents of an assembly for the purpose of reverse-engineering and to support plug-ins. I guess you could use it in place of the Introspection API, but for static analysis I'd use the Introspection API. There are other APIs/libraries out there to pick apart assemblies. One was similar to BECL - I think it was called MCEL - but I can't find a link to it. I mentioned two others nearly 4 years ago, but the ILReader doesn't seem to be around any more and CLIFileReader hasn't had any activity in nearly 4 years.

Update (3/23/2006) - I just saw this API: the PERWAPI. I haven't tried it out personally, but I perused the documentataion (PDF) and it seems pretty thorough (although I feel uncomfortable with their design choice relative to having a CLS-compliant interface).

So, in a nutshell, if you need to do dynamic invocation, use Reflection. If you just need to read the contents of an assembly, use the Introspection API. In fact, if you're going to make an Object Browser, I'd argue that you shouldn't use Reflection if your intent is just to browse (i.e. read-only). I just wish the Introspection APIs would have more exposure than what they have - they're pretty powerful.

[1] For a while, the Phoenix site didn't have a lot of information. Now it's been beefed up a bit - you can actually get the RDK for non-commercial use, and there's some sample information available. Pretty cool - I gotta check this out!

[2] That's one thing that sucked about being sick yesterday - I missed out on a Phoenix talk, and I had a bunch of questions that I wanted to ask!

> Published: 03.16.2006 11:15:56 AM CST