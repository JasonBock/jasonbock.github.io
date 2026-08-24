---
title: This Feels Like Cheating
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# This Feels Like Cheating

As I was trying to add my messaging page to my blog engine, I wound up needing a helper method to make the invocation easier for the UI. After futzing with it for a second, I realized that generics makes me "get away" with having a generic return value. Look at the following code:

```c#
public T Get() where T : class, new()
{
  T returnValue = Activator.CreateInstance();
  return returnValue;
}
```

This allows me to write the following code:

```c#
MessageRequest newRequest = this.Get();
MessageResponse newResponse = this.Get();
```

This isn't the exact scenario I'm running into, but what floored me is that I don't have to have a bunch of `Get()` methods that are all named different since you can't overload a method in C# just by a different return type (you can at the CIL level). Rather, I can use a generic method. Granted, there are some restrictions I can put into place, but this technique allows me to write one helper method.

Generics are so awesomo.

> Published: 04.11.2006 07:17:45 PM CST