---
title: And I Though Putting an Underscore in Front of Field Names Was a Good Idea ...
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# And I Though Putting an Underscore in Front of Field Names Was a Good Idea ...

My client has a bunch of coding standards that I have to follow for my project. Nothing major - most of the common sense stuff that you would see in any good coding standard. One of the standards is to attribute your assembly with `CLSCompliant`, like this:

```c#
[assembly: System.CLSCompliant(true)]
```

All fine and good, but I'm used to having an underscore ("_") in front of my field names, like this:

```c#
public abstract class MyBaseClass
{
  protected string _myData = string.Empty;
}
```

The problem with this is that the underscore isn't CLS-compliant:

```
"C:\somedirectory\somefile.cs(23): Identifier 'MyBaseClass._myData' is not CLS-compliant".
```

I've never run into this before as I haven't attributed my classes with this attribute before (although I was definitely aware of it). This kind of stinks because I liked being able to type "_" and then do a Ctrl+space key combo in VS .NET to make it easier to get the right field name in my code. Oh, well, life could be worse. I could be coding in VB .NET (just adding fuel to all the recent flame wars over languages!)

> Published: 07.14.2004 10:41:51 PM CST