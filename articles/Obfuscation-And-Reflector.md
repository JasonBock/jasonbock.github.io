---
title: Obfuscation and Reflector
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Obfuscation and Reflector

As some of you know, I've created a tool called FileGenerator - a Reflector add-in that would generate code files. Well, to make a long story short, Reflector is now obfuscated with version 3.4.7.0, and I can no longer figure out which of the obfuscated classes are the `Optimizer` and `Translator` classes (although I have a hunch that `Translator` is now _43, but I'm not sure). Believe me, I tried, but no dice. I **could** keep digging around in the API, but I won't. It's just not worth my time and effort.

When I first created my add-in, all of the classes I used were public, but they were not documented. Therefore, it took a bit of spelunking to figure out what I needed to do, but it was definitely achievable. I'll rate this level of complexity at a 1. Some time after I released my add-in, these classes became non-public, but I could still see the classes in ILDasm or Reflector. Therefore, a bit of Reflection code was needed to get my add-in back online. Now the complexity level went up about an order of magnitude, or 10. I could still get my code to work, but it was more fragile. Once Reflector became obfuscated, all bets were off. Even if none of the class definitions changed in 3.4.7.0, any future changes would be very difficult to track. In my view, the complexity factor shot up another order of magnitude (100) which is beyond my pain threshold. It used to be interesting keeping my add-in operational; now it's extremely annoying.

Therefore, I'm done updating FileGenerator. It's unfortunate that a public API in a free tool has been obfuscated, but I don't own that source code, so I'm not going to spend any more time trying to figure out how my add-in should work with future versions of a tool that I can have little to no influence over its design [1]. It would be interesting if someone in the .NET community started an open-source, decompilation project with a plug-in, extensible architecture. I have to admit that I've never done any work in the area of decompilation, so I know I'm not the one to lead it, but I'd definitely try to be involved in testing and design. It's been fun using other open-source .NET tools over the years, like Nant and NUnit, and it would be cool if something like NSource would show up on the scene.

[1] Just to be clear, I like Reflector. It's a nice tool that complements ILDasm.

> Published: 03.15.2004 09:46:55 PM CST