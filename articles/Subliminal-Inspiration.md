---
title: Subliminal Inspiration
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Subliminal Inspiration

A couple of days ago, I added this class to our code base:

```c#
using System;

public static class EnumParsing
{
  public static T? TryParse<T>(string name, bool ignoreCase)
    where T : struct
  {
    T? value = null;

    if(!string.IsNullOrEmpty(name))
    {
      Type tType = typeof(T);

      if(tType.IsEnum)
      {
        try
        {
          value = new T?((T)Enum.Parse(tType, name, ignoreCase));
        }
        catch(ArgumentException)
        {
        }
      }
    }

    return value;        
  }
    
  public static T? TryParse<T>(string name)
    where T : struct
  {
    return EnumParsing.TryParse<T>(name, false);
  }
}
```

The intent is you can parse an enumeration value as a string without throwing an error. It's type-safe, and it also returns null if the conversion didn't work. I did this on a whim - I didn't think C# would let me return a nullable type like that, but ... it did. Nice.

> Published: 09.24.2007 11:54:27 PM CST