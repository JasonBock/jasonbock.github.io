---
title: The Benefits of Code Analysis (Some of Which are Unexpected)
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Benefits of Code Analysis (Some of Which are Unexpected)

## Abstract
Analyzers in .NET provide automatic reviews of your code. They can find issues and, in some cases, offer a code fix to address the problem. This article will show how analyzers were used to review the Rocket Libraries codebase and how one suggestion led to an unexpected (and beneficial) refactoring.

## Introduction

As software developers, we build applications that delight users with their intuitive interfaces and wonderful performance. Unfortunately, we also make mistakes. I've been writing code for more than 25 years, and I've never known a developer that has consistently created error-free code.

The severity of these mistakes varies. Sometimes, they're minor without causing too much damage, but in other cases they can have serious repercussions. Unfortunately, it doesn't take much to create a catastrophic problem. Case in point: Cloudflare's parser bug colloquially known as [Cloudbleed](https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/). One line of code was the root cause of the issue which led to a significant monetary loss for Cloudflare users.

Code review processes play a crucial role by reducing bugs in our code. Traditionally, code reviews happened in a conference room, where developers would review each other's changes and provide suggestions to improve them before merging them into the main branch of a code repository. These days, [GitHub](https://github.com/) makes reviewing code easier by enabling developers to evaluate [requests](https://docs.github.com/en/pull-requests/reference/pull-requests) (PR) and provide comments as feedback in the tool.

Manual code reviews like those in conference rooms and on GitHub still rely on humans. We can miss bugs even when they're right in front of us on our screen. The Cloudbleed bug existed for years before anyone noticed it. If you're a C# developer and saw the code in Listing 1, would you know what the problem is?

*Listing 1 – Problematic Code With No Context*

```c#
var service = new CustomerService();
```

We'll come back to this later, but don't feel too bad if you can't figure it out - that's the point. Sometimes we don't have enough context to spot an issue.
In instances like this, code analysis tools come in handy. Code analysis tools automatically scan code for known issues and alert a developer as soon as they're coding so they can quickly remedy it.

In .NET, analysis tools come in the form of analyzers. These analyzers will create diagnostics based on violations of a particular coding rule, and in some cases, offer a code fix such that a developer can easily resolve the issue. The analyzers and code fixes themselves are authored by a developer - in a future article, I'll cover how to create custom analyzers to meet your project's needs.

## Project Setup

New C# projects that target .NET 6 will automatically enable the [Roslyn Analyzers](https://github.com/dotnet/sdk). It doesn't matter what kind of project it is - this will work for console applications, class libraries, and so on. (For projects that target older .NET framework versions, you can reference [this package](https://www.nuget.org/packages/Microsoft.CodeAnalysis.NetAnalyzers) in NuGet.) I recommend you change the two project settings to amplify the analysis: [`<AnalysisMode>`](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props#analysismode) and [`<TreatWarningsAsErrors>`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/errors-warnings#treatwarningsaserrors).

By default, `<AnalysisMode>` is set to `Default`, which means that some of the rules are disable. I recommend you change this setting to `All` for a more aggressive analysis - the more problems that analyzers can find for you, the better.
 
By default, `<TreatWarningsAsErrors>` is `false`, but I recommend setting it to `true`. If you want to know whether your code has issues, your build needs to break. Otherwise, you're relying on developers sifting through warnings to determine which ones they'll fix. In my experience, if a compilation is successful (regardless of warnings), developers will commit their changes and move on. Using the analyzer to display warnings as errors will force you to resolve what's going on in the code. Furthermore, you can disable specific analyzers in an .editorconfigfile. I'll show you how that works in the next section.

Keep in mind, if you haven't changed these values before on existing projects, changing them can be jarring. Your build will likely fail, and you'll see a few issues reported that you hadn't seen before.

Set these values as soon as possible for new projects. You may want to set aside some time to address new issues revealed by the analyzers in existing projects. Either way, these analyzers will find things that the standard compiler won't report. The sooner you get used to this flow, the better off you'll be. You should consider using a [Directory.Build.props](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-your-build) file for your solutions with these values set in the file. That way, all your projects will have the desired configuration.

If you're using Visual Studio, there's value in perusing the *Error List* window and reviewing the warning and information messages, as seen in the following screen shot.

*Figure 1: The Error List in Visual Studio*

![Benefits CodeAnalysis](https://jasonbock.net/images/Benefits-CodeAnalysis-1.png "Benefits CodeAnalysis")

Visual Studio also has analyzers that provide insight into your code. Most of the time, these are intended to be helpful suggestions and not to break your build. You may want to enforce some of these to ensure all developers on a project adhere to the same standards. Through settings in your .editorconfig file, you can raise your awareness of these insights by changing the analysis level of that analyzer. I'll cover how to modify your .editorconfig in the next section.

Many other analyzer packages from Microsoft help with API creation and consumption, like [Microsoft.CodeAnalysis.PublicApiAnalyzers](https://github.com/dotnet/roslyn-analyzers/blob/main/src/PublicApiAnalyzers/PublicApiAnalyzers.Help.md) and [Microsoft.CodeAnalysis.BannedApiAnalyzers](https://github.com/dotnet/roslyn-analyzers/blob/main/src/Microsoft.CodeAnalysis.BannedApiAnalyzers/Microsoft.CodeAnalysis.BannedApiAnalyzers.md). Furthermore, several analyzer packages look at code for specific cases. For example, [NUnit.Analyzers](https://github.com/nunit/nunit.analyzers) looks for issues in tests written with [NUnit](https://nunit.org/).

## Reviewing Analysis Results

Once you've made the appropriate changes in your project setup, you'll probably start to see issues light up during compilation and in your integrated development environment (IDE). Let's review a few analyzers and code fixes that you might see.

The first rule the analyzer may flag is [CA1829](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1829), which occurs if you're using the Count()extension method on an enumerable value, as shown in Listing 2.

*Listing 2 – Using LINQ Count() Extension Method*

```c#
public static class UseLinqCount
{
  public static int GetNameCount => UseLinqCount.GetNames().Count();

  private static List<string> GetNames() => new() { "Joe", "Jane" };
}
```

Fortunately, this analyzer has a related code fix to update your code automatically. In Visual Studio, you can press *Ctrl+{period}* to light up the code fix.

*Figure 2: Reviewing a code fix from an analyzer*

![Benefits CodeAnalysis](https://jasonbock.net/images/Benefits-CodeAnalysis-2.png "Benefits CodeAnalysis")

Notice also that you can fix all occurrences in the current document, the current project, or all code files within the entire solution. Of course, not all code fixes have this update capability, but if they do, this kind of batch fixing can save you a lot of time by fixing the issue everywhere it appears.

Another rule the analyzer may flag is [CA1823](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1823), which happens if you have a field in a class that isn't being used, as shown in Listing 3.

*Listing 3 – Unused Fields in Code*

```c#
public sealed class CodeAnalysisUnused
{
  private Guid used;
  private int x;

  public CodeAnalysisUnused(Guid used) =>
    this.used = used;
  
  // ...
}
```

It's common for projects with a lot of history to end up with unused members like this. So other analyzers within Visual Studio will also let you know when a parameter isn't being used in a method, like rule [IDE0060](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0060):

*Listing 4 – Unused Parameters in Code*

```c#
public static int Add(int a, int b, int c) =>
  a + b;
```

In this method, the `c` parameter isn't being used by `Add()`. However, you must be careful in removing the parameter in a case like this. If this API has already been shipped and used by customers, removing the parameter would break their code.

Sometimes, .NET adds members that you should use instead of previously existing APIs. For example, let's say you had the following call to Contains ()

*Listing 5 – Calling `Contains()` on a `String` Object*

```c#
public static class StringRules
{
  public static bool HasCharacter(string value) =>
    value?.Contains("I", StringComparison.OrdinalIgnoreCase) ?? false;
}
```

In .NET 6, an overloaded version of `Contains()` accepts a `char` as the first parameter. So, if you're currently passing in a one-character length string to `Contains()`, the [CA1847](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1847) rule will let you know you should change it to a char. This rule also has a code fix to make it easy to update.

While I find rules like this helpful in improving a codebase, there are some that I'd rather not use. For example, [CA1014](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1014) will tell you to mark your assembly with the `CLSCompliantAttribute`. However, I don't find value in this attribute, so I'll ignore this one with an entry in my .editorconfig:

*Listing 6 – Ignoring Specific Analysis Rules*

```
dotnet_diagnostic.CA1014.severity=none
```

Doing so will prevent the analyzer with the given code from showing up in your error list.

Other times, I want to follow some rules with new code, but implementing the rule means possibly breaking existing APIs. An example of this is a property that returns an array is represented in Listing 7.

*Listing 7 – Returning an Array From a Property*

```c#
public sealed class UsingArrayForProperty
{
  public string[] Values { get; set; }
}
```

The [CA1819](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1819) rule will flag this property with an issue because the code should return a copy of the array. However, if this type has already shipped, you may not want to risk breaking users of the type. Therefore, you can put a #pragma warning around the property as shown in Figure 3 to disable the rule for new code.

*Figure 3: Disabling an analyzer issue*

![Benefits CodeAnalysis](https://jasonbock.net/images/Benefits-CodeAnalysis-3.png "Benefits CodeAnalysis")

For exceptional cases like this, I find suppressing specific rules acceptable. However, if you find that you're suppressing a specific rule continuously, you may want to consider configuring it in your .editorconfig so it won't show up at all.

## Unexpected Benefits

I've included analyzers in my .NET projects since I heard of [FxCop](https://learn.microsoft.com/en-us/visualstudio/code-quality/net-analyzers-faq) (the original incarnation of code analysis in .NET), and it's helped me write better code with fewer mistakes. Of course, they don't catch everything, but they find things that I would've missed. They can also have unexpected benefits.

Rocket Libraries, my team at Rocket Mortgage, is responsible for creating reusable components that address cross-cutting concerns across multiple teams. We've modernized the packages used by developers across our Technology team by only targeting supported frameworks, using current language features and other improvements. We're also adding analyzers in our projects, and as you might expect, the results are helping us find ways to improve our code (or, as we say at Rocket, ["finding the inches"](https://www.rocketcompanies.com/who-we-are/our-philosophies/#our-isms)).

For example, we uncovered places in our code where we used objects for locks that didn't have the right type (rule [CA2002](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2002)). In most cases, it was relatively straightforward to address the issues found by the analyzers. However, these changes helped us notice other areas we could improve in our code.

Here's an example. We had something like this in one of our libraries:

*Listing 8 – Returning an Enumeration From a Method*

```c#
public IEnumerable<string> GetData()
{
	var data = new List<string>();
	data.Add("a");
	return data;
}
```

The actual code is a bit more complicated, but this code snippet captures the essence of the problem. When I was reviewing the library, I was using an .editorconfig that I created for open source projects, which has this entry:

*Listing 9 – Rule Configuration in .editorconfig*

```c#
dotnet_style_collection_initializer = true:error
```

I prefer having similar code styles for similar projects and having them configured to a level where a build will fail, which enforces the styles to be consistent. In this case, it flagged this piece of code as an issue. Fixing it is simple, especially with the code fix built into this analyzer.

*Figure 4: Using a code fix for collection initialization*

![Benefits CodeAnalysis](https://jasonbock.net/images/Benefits-CodeAnalysis-4.png "Benefits CodeAnalysis")

When I saw this flagged, I looked at the method itself. It was only being called in two locations within the containing class (the method was marked as `private` in the actual code). There was a `First()` LINQ call on the return value in both cases. I realized that we should change the return value to the data type itself. We don't need to return an array, list or anything other than the actual value. This change eliminated a call to `First()` and slightly improved our code's performance.

Being forced to look at existing code helped us uncover unexpected but related issues. So, even if you disagree with the analyzer, it might show you other issues you wouldn't have caught under normal circumstances.

## Conclusion

Using automatic code analysis for your .NET applications provides you with a simple, automated way to find (and sometimes even fix) issues in your code. If you're not using them, I encourage you to start reading what's in your compiler's error list. You may be surprised by how many errors it finds in your codebase! If you have any questions, please leave a comment to this article. Finally, keep watch for part two of this article, where I'll show you how to create an analyzer and a code fix.

You can find the code samples in this article from [this repository](https://github.com/jasonbock/AnalyzingCodeInDotNet).

> Published: 07.07.2021