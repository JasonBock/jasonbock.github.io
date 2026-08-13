---
title: Not Everything Has a DeclaredType
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Not Everything Has a DeclaredType

Since my client has the day off and it snowed like crazy yesterday (so a lot of shoveling has been done) I decided to take 1/2 the day off. So why not spend a little time blogging about another case of "just because you have unit tests around code, doesn't mean it's right."

Take a look at this code snippet:

```c#
return string.Format(CultureInfo.CurrentCulture, 
  "[{0}], {1}::{2}({3})",
  targetMethod.DeclaringType.Assembly.GetName().Name,
  targetMethod.DeclaringType,
  targetMethod.Name, paramBuilder.ToString());
```

This is from my [Spackle](https://github.com/JasonBock/SpackleNet) library. It's part of the code that gives you a nice summary of exception information in a `TextWriter`-based object. This code takes a `MethodBase` and returns a detailed description of it.

There's a problem here though. What happens if the method that either threw the exception or is in the stack trace is a `DynamicMethod`?

(I'll give you a minute to figure it out)

Stumped? Looked at the Reflection API (or on teh internets)? Or you knew what the issue was right away?

OK, here's the problem. With a `DynamicMethod`, there is no `DeclaringType` - it's `null` (as it clearly says [in the SDK](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.emit.dynamicmethod.declaringtype)).

Oops.

The [5.1 release](https://www.nuget.org/packages/Spackle/5.1.0) has a fix for this.

It's interesting to note that the `Source` for an exception that's thrown from a `DynamicMethod` is "Anonymously Hosted DynamicMethods Assembly". Which makes sense - that dynamic code has to go somewhere!

By the way, I'm not putting down unit tests in the first paragraph. Unit tests are **extremely** beneficial. But they don't always catch everything!

> Published: 02.21.2011 10:37:50 AM CST