---
title: Enumeration Rumination
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Enumeration Rumination

I now interrupt this blog with two things I'd like to talk about with respect to enumerations.

The first one deals with method arguments and booleans. Take a look at this code:

```c#
this.Build(true);
```

So, can you tell me what `Build()` does? Other than build something?

How about this:

```c#
this.Build(Directories.IncludeSubdirectories);
```

This reads easier. You know that `Build()` has something to do with files and directories and so on. Now, I know, you'll probably counter with, "are you kidding me? change all my method arguments that are booleans to enumerations?" Enumerations give you a bit more flexibility as you can add more values in the future if need be without having to change the signature, and they provide more information in the code. I'm not saying you should change all your method signatures, but give this idea some thought in the future.

Speaking of adding enumeration values ... I just found out there's an exception in .NET called `InvalidEnumArgumentException`. You may want to use it in the default section of a switch statement:

```c#
public static Beer Create(Manufacturer manufacturer)
{
  Beer beer = null;

  switch(manufacturer)
  {
    case Manufacturer.Newcastle:
      beer = new Newcastle();
      break;
    case Manufacturer.Duff:
      beer = new Duff();
      break;
    case Manufacturer.Blatz:
      beer = new Blatz();
      break;
    default:
      throw new InvalidEnumArgumentException();
  }

  return beer;
}
```

Enumerations ... so quirky ... yet so useful ... sometimes.

> Published: 05.19.2009 11:21:39 PM CST