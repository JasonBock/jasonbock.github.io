---
title: One Thing You Can't Do With Generics in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# One Thing You Can't Do With Generics in .NET

I thought I'd try to be creative tonight and try this in C# 2.0 [1]:

```c#
public class BaseClass 
{
  public int GetTypicalAnswer { return 42; }
}

public class AddMethod<BaseType> : BaseType 
{
  public int GetWackyAnswer() { return 1001001; }
}

public class TestAddMethod
{
  public void Test()
  {
    AddMethod<BaseClass> extendedBaseClass = 
      new AddMethod<BaseClass>();
    int typicalAnswer = extendedBaseClass.GetTypicalAnswer();
    int wackyAnswer = extendedBaseClass.GetWackyAnswer();
  }
}
```
But such wackiness isn't allowed:

```
Error 1: Cannot derive from 'BaseType' because it is a type parameter
```

Oh well. It's no big deal, but, man, having this would open up so much wackiness!

[1] I've noticed that most of the .NET generic classes use the one-letter capitalization approach for type names, like `Dictionary<K, V>`. Personally, I like more descriptive variable names (and isn't that what the generic type name is anyway? At least, sort of?), like `Dictionary<Key, Value>`. A small issue, but I know some coding standard war will start up on this someday.

> Published: 10.20.2004 06:26:57 PM CST