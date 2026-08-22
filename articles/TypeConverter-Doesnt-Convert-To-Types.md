---
title: TypeConverter Doesn't Convert To Types
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# TypeConverter Doesn't Convert To Types

For some reason, the fact that the following code throws a `NotSupportedException` ('TypeConverter' is unable to convert 'System.String' to 'System.Type') made me laugh:

```c#
using System;
using System.ComponentModel;

namespace TypeConversion
{
  class Program
  {
    static void Main(string[] args)
    {
      TypeConverter converter = new TypeConverter();
      Type convertedType = (Type)converter.ConvertTo(
        "TypeConversion.Program, TypeConversion", typeof(Type));
    }
  }
}
```

> Published: 03.26.2007 09:43:50 AM CST