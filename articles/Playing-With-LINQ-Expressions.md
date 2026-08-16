---
title: Playing With LINQ Expressions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Playing With LINQ Expressions

At the upcoming Iowa Code Camp, I'm going to do 2 talks [1], one on exceptions and one on Reflection. Based on a couple of suggestions I'm updating the material to the Reflection talk slightly - one of the things I'm adding is a demonstration of using LINQ expressions to create a dynamic method. It's kind of a brain-twister to me - it took some help from Aaron to help me see some of the light and I still don't quite grok the whole thing, but it's quite cool. Of course, what I'd really love to see is something like this:

```c#
var method = MethodCompiler.Compile(
  "int DoSomething(MyThing thing)" +
  "{" +
  "    int result = 0;" +
  "    try" +
  "    {" +
  "        result = thing.X / thing.Y;" +
  "    }" +
  "    catch(DivideByZeroException)" +
  "    {" +
  "        result = 22;" +
  "    }" +
  "    return result;" +
  "}";

int result = method.Invoke(myThing);
```

Maybe in .NET 5.0 with the whole "compiler as a service" idea, huh?

The odd thing is that my timing tests shows that the expression version is just a tad faster than my `DynamicMethod` version. I really wish I could see the IL of the compiled expression, but I can't find a way to do it. At least that way I'd be able to see if there's any differences in the generated IL.

[1] At least I think I am ... the site doesn't show my 2nd talk as of yet, but I was told I'm doing two talks.

> Published: 11.05.2008 10:14:46 AM CST