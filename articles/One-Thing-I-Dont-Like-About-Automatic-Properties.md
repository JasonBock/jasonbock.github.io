---
title: One Thing I Don't Like About Automatic Properties
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# One Thing I Don't Like About Automatic Properties

I glanced at a newsgroup post in my favorite RSS reader, and for some reason that one sentence prompted my brain to rant a bit on automatic properties.

Basically, automatic properties look like this:

```c#
public sealed class Properties
{
  public Guid Id
  {
    get;
    private set;
  }
}
```

What C# does behind the scenes for you is create a private field with a mangled name - it looks like this (copied straight from Reflector):

```c#
[CompilerGenerated]
private Guid <Id>k__BackingField;

public Guid Id
{
  [CompilerGenerated]
  get
  {
    return this.<Id>k__BackingField;
  }
  [CompilerGenerated]
  private set
  {
    this.<Id>k__BackingField = value;
  }
}
```

Basically, the field name is the name of the property surrounded with brackets and appended with the "k__BackingField" string.

For simple read/write properties, automatic properties do the job. But the issue I have is that it's inconsistent. Namely, if I need to change how the getter or setter works, I can't use automatic properties anymore; I have to resort to writing the logic myself:

```c#
public sealed class Properties
{
  private Guid settableId;
    
  public Guid Id
  {
    get;
    private set;
  }
    
  public Guid SettableId
  {
    get
    {
      return this.settableId;
    }
    set
    {
      if(value == Guid.Empty)
      {
        throw new ArgumentException("The ID cannot be empty.", "value");
      }
            
      this.settableId = value;
    }
  }
}
```

Wouldn't it be way cool if C# would have a keyword that would represent the backing field? Then you could do this:

```c#
public sealed class Properties
{
  public Guid Id
  {
    get;
    private set;
  }
    
  public Guid SettableId
  {
    get;
    set
    {
      if(value == Guid.Empty)
      {
        throw new ArgumentException("The ID cannot be empty.", "value");
      }
            
      this.autofield = value;
    }
  }
}
```

That feels like the best of both worlds. You can use the standard get/set implementations, but if you want to customize one or the other (or both), then you're free to do so. This pretty much eliminates the need for the developer to explicitly define fields that backs a property. Maybe "autofield" isn't the best syntax, but you get the point - have a keyword that represents the backing field in a simplistic way.

I kind of like this, but it's a Friday and I haven't had enough coffee to think this through, so there may be gaping holes in my design. But I do hope automatic properties are updated in a future version of C#.

> Published: 10.03.2008 07:37:02 AM CST