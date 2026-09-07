---
title: CodeDOM Question: How to Set Up the `CompilerParameters` Object?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# CodeDOM Question: How to Set Up the `CompilerParameters` Object?

I'm doing some work with the CodeDOM classes that augments the C# compiler. Basically, I'm going to tweak the code based on some conditions, and then invoke the real C# compiler. Here's my problem. I want the tool to take the exact same command-line arguments that the C# one does, like this:

```
myCSharpCompiler /target:library File.cs
```

The problem is the `CompileAssemblyFromFileBatch()` method (or any `ICodeCompiler` method for that matter). The first argument is a `CompilerParameters` object. It looks like you need to set values like `GenerateInMemory`, `ReferencedAssemblies`, etc. I could go through the command-line args passed in `Main()` and set each property appropriately, but I'm wondering if there's a method that will take a string array and produce a `CompilerParameters` object that has all of the properties correctly. There is a `CompilerOptions` string property on `CompilerParameters` - it looks like that it can take all of the arguments in one string, but I'm not sure if it'll set all of the other props correctly (i.e. I set `CompilerParameters` to "/target:winexe" so the output will be an EXE, but what happens to the `GenerateExecutable` property's value?).

There is a method called `CmdArgsFromParameters()` that's on the `CodeCompiler` class, but it seems to do the reverse of what I'm looking for. What I'd like is a method in reverse: `ParametersFromCmdArgs()` that takes either a `string` or a `string` array and returns a `CompilerParameters` object. Does something like this exist? Any help is appreciated - thanks in advance.

> Published: 04.16.2003 02:48:45 PM CST