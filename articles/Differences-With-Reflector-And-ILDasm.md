---
title: Differences with Reflector and ILDasm
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Differences with Reflector and ILDasm

Usually, my under-the-hood-of-.NET tool of choice is Reflector. Just recently, though, I ran into something that looked a little odd, so I had to use my 2nd-favorite tool of choice, ILDsam. Here's the situation - I have the following, simple class:

```c#
public class AClass
{
  private int value;

  public AClass(int aValue)
  {
    this.value = aValue;
  }
}
```

Now, here's what it looks like in ILDasm:

![Code in ILDasm](https://jasonbock.net/images/Reflector-ILDasm-1.png "Code in ILDasm")

No surpises there. Now take a look at Reflector:

![Code in Reflector](https://jasonbock.net/images/Reflector-ILDasm-2.png "Code in Reflector")

What strikes me as odd is that I'm seeing a no-argument constructor, methods like `FastGetExistingType()`, and so on. The reason they're showing up is because I have "Show Inherited Members" turned on under "View -> Options":

![Reflector Configuration](https://jasonbock.net/images/Reflector-ILDasm-3.png "Reflector Configuration")

But I thought constructors weren't inherited? And what's also weird is that things like `Equals(object, object)` are showing. Static methods aren't stricly "inherited". I see that there's a "{ System.Object }" tag at the end of the method, though. Maybe the method tree view can be split into "Static" and "Instance" nodes. I don't know ... it's not a big deal but it just surpised me to see constructors and static methods show up from the base class.

> Published: 03.16.2006 02:33:08 PM CST