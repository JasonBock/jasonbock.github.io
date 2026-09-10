---
title: Rocking Your Mocks
layout: default
---
| [Home](https://jasonbock.NET/index.html) | [Biography](https://jasonbock.NET/Biography.html) | [Speaking](https://jasonbock.NET/Speaking.html) | [Articles](https://jasonbock.NET/Articles.html) | [Books](https://jasonbock.NET/Books.html) | [Music](https://jasonbock.NET/Music.html) |

# Rocking Your Mocks

## Introduction

Up until .NET 4.6, the primary way to create new code at runtime has been to emit IL via the `System.Reflection.Emit` namespace. This requires deep knowledge of IL, a skill most .NET developers don't have, and it's painfully hard to debug emitted assemblies this way as there's no textural code that's generated with the assembly. Now, with the Compiler APIs in .NET 4.6, you can easily generate code in C# or VB, execute it on the fly, and load that new code into a debugger. In this article, I'll show you how this is done with [Rocks](https://github.com/JasonBock/Rocks), my new mocking library for .NET.

## Simplifying Code Generation

I like code generation. Whether it's part of a build process or done at runtime, seeing application use code that was created via a generative process is exciting to me. Generalizing the development process is a good thing in my mind – if we can figure out how to automate some percentage of implementation, we can move on to other, potentially more interesting problems. Granted, code generation can bring forth its own set of problems, but if used right it can free developers up from having to deal with the boring, tedious stuff.

In .NET, there's been a number of ways to generate code at runtime. The long-time standard is to use `System.Reflection.Emit`, especially if you need to generate an entire class. The problem with this approach is that it requires a developer to know how the Intermediate Language (IL) works. This is the "assembly code" for all .NET languages, and while it's not as difficult as x86 assembly, it's also not as clear-cut as C# is. I've [written a framework](https://github.com/JasonBock/DynamicProxies) that creates proxies at runtime and I know from experience that it's not easy to generate a valid assembly. In fact, I ended up [writing another framework](https://github.com/jasonBock/AssemblyVerifier) that uses peverify so I know right away if my assembly was riddled with bugs that do not show up with code written in C#.

Another problem with this approach is debugging. Even if my generated assembly is verifiable, it may still have logic bugs in the code. However, it's very difficult to debug these assemblies as there isn't any source code generated that can be mapped to symbolic debugging information. I [wrote a framework](https://github.com/jasonBock/EmitDebugging) to do this, and it works, but it's kind of clunky.

What I really want is to just write C# code, compile it, and run it. In the past, I could have used csc.exe, but now there's a much easier and streamlined way to do it via the Compiler API (https://github.com/dotnet/roslyn/). Furthermore, it also makes it easy to debug any generated code you create. Let's see how this works.

## Using the Compiler API to Create Mocks

I'm a big fan of unit testing. I'm also a big fan of isolation in unit testing. If you can provide mocks to code under test rather than using a number of monolithic dependencies, I think you're in a good space. I've used [Moq](https://github.com/Moq/moq4) and [NSubstitute](http://nsubstitute.github.io/) before and they're great libraries. What I wanted to do is see if I could accomplish the same thing using the new Compiler API assemblies in .NET 4.6. That's what [Rocks](https://github.com/jasonBock/rocks) is. You give it an interface or an unsealed class – something that contains virtual members – and then you can set up expectations with that mock. Once that's done, you pass it to an object that will use it in a test, and you can verify that it was used as expected.

I'm not going to go through every scenario that Rocks handles. There's a [quickstart available](https://github.com/JasonBock/Rocks/blob/main/docs/Overview.md) if you want to get more details. What I'll do in this article is go through a simple example of how a mock is created, and how I can debug the resulting mock.

Let's say you have an interface defined like this:

```c#
public interface ITest
{
  void Run();
}
```

To create a mock and verify its usage, here's what you do:

```c#
var rock = Rock.Create<ITest>();
rock.Handle(_ => _.Run());

var chunk = rock.Make();
chunk.Run();

rock.Verify();
```

It's pretty easy, but let's see what's going on underneath the scenes. And by that I mean, let's not take a look at the code within Rocks; that's all within the repo. Let's take a look at the code that's generated for the mock. To do that, you need to change one line of code:

```c#
var rock = Rock.Create<ITest>(
  new Options(level: OptimizationSetting.Debug, 
    codeFile: CodeFileOptions.Create));
```

By default, Rocks uses release settings and it doesn't create a code file. But by changing the options, you'll not only save the code to a file on disk, but you can step into the generated code. Here's a screen shot of Visual Studio 2015 with `Run()`'s method implementation in a debugging session: 

(2026 - sorry, the image is gone)

You can see that a class with a long, mangled name is created that implements `ITest`. You can also see that a dictionary of method tokens to classes called `HandlerInformation` is used to ensure that a member is used exactly as the expectations were set up.

I don't expect users of Rocks to step into a mock. That feature is primarily one for me as the designer of Rocks. But I have it there as an option for anyone to use as it may be valuable to use if a user runs into a bug. They can turn debugging on, step into the code and see where it's failing (and potentially why). This can make bug fixing much easier than with IL generation. I used this all the time when I was trying to create mocks for all the mockable types in mscorlib. There were a number of cases that I wasn't handling correctly that showed up with methods and properties in mscorlib, and being able to get clear compilation errors, plus generated source code to see where the error was, was incredibly helpful.

## Feedback, Please!

It's been fun to create this library, and while one of my goals was to create a working example of run-time Compiler API-based code generation, I'd like to see this used in real-world unit testing projects as well. If you run into any bugs or think of any enhancements, please submit them to the Github page. Also, if you'd like to contribute, I'd love to distribute the issue load with others. Just let me know and I'll have you submitting pull requests in no time!

> Published: 08.25.2015 01:51:00 PM