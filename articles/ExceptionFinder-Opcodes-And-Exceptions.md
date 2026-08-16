---
title: ExceptionFinder, Opcodes, and Exceptions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# ExceptionFinder, Opcodes, and Exceptions

It's funny how things sometimes work together ...

One of the things I demonstrate in my exceptions talk is the checked keyword in C#. This basically emits an add.ovf opcode, rather than an add opcode. add.ovf will throw an OverflowException if the operation ... overflows. Now, later on in the talk I demonstrate my [ExceptionFinder](https://github.com/JasonBock/ExceptionFinder) add-in for [Reflector](https://www.red-gate.com/products/reflector/), and I showed all of the exceptions that can be thrown from one of the constructors on `Guid`. It's always bothered me that the documentation says it can throw an `OverflowException`, but my analysis never found it.

Then my brain woke up: I realized that opcodes can also throw exceptions, but I wasn't including that in my analysis. As soon as I realized this, I felt really stupid, but at least I remembered it!

So ... I'm currently adding some code to the add-in to check each opcode and add exceptions to the list that the Partition docs says the opcode might throw. This is more work than I thought, but it's needed. It would've been really nice (hint, hint, MS BCL documentation writers!) if the exceptions listed in the Partition docs were also listed as `<exception>` elements in the XML comment section. That would've saved me a ton of work because then I wouldn't have to add that all in myself. The exceptions are listed in the .NET docs, but they're not listed explicitly.

Anyway, I'm hoping that when I get this all in place I'll finally have `OverflowException` in the `Guid` constructor listed as a documented and found exception.

> Published: 10.13.2008 11:09:01 AM CST