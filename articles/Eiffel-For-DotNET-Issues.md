---
title: Eiffel for .NET Issues
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Eiffel for .NET Issues

More problems with Eiffel. Class invariants aren't being evaluated in my assembly. The code for my invariant ends up in the `_invariant()` method as expected, but it's never evaluated. Also, adding a precondition to a feature doesn't work - the assertion code shows up before the main method's code, but it's never evaluated. I also noticed the compiler sometimes adds `castclass` opcodes when it doesn't need to. For example, a method may take an object reference of type `MyClass`, and then the CIL will try to cast that reference to `MyClass`. I can't figure out why this cast is being done - it seems unnecessary. I also started to get strange compilation errors after I commented two lines of code out. The only way I could get a release build working again was to close VS .NET, rename the current release directory, move the ACE file to a new release directory, and re-open the project. Finally, I've noticed that modifications to my Eiffel code don't show up in the assembly until I close the entire VS .NET session and re-open the project.

My feeling about Eiffel for .NET is that the language is cool, but the current 1.0 implementation is lacking in areas.

> Published: 11.05.2002 11:46:00 PM CST