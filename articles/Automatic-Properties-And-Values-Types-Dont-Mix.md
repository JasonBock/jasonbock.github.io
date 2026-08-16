---
title: Automatic Properties and Values Types Don't Mix
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Automatic Properties and Values Types Don't Mix

Consider the following struct:

```c#
public struct BirthDate
{
  private DateTime value;
    
  public BirthDate(DateTime value)
  {
    this.value = value;
  }
    
  public override string ToString()
  {
    return this.Value.ToString();
  }

  public DateTime Value
  {
    get
    {
      return this.value;
    }
  }
}
```

This works just fine.

Now let's make our life easier by refactoring to use automatic properties:

```c#
public struct BirthDateWithAutomaticProperties
{
  public BirthDateWithAutomaticProperties(DateTime value)
  {
    this.Value = value;
  }

  public override string ToString()
  {
    return this.Value.ToString();
  }

  public DateTime Value
  {
    get;
    private set;
  }
}
```

Nope, this won't compile:

```
Error 1 The 'this' object cannot be used before all of its fields are assigned to.
```

OK ... let's try this:

```c#
public struct BirthDateWithAutomaticProperties
{
  public BirthDateWithAutomaticProperties()
  {
    this.Value = DateTime.Now;
  }
    
  public BirthDateWithAutomaticProperties(DateTime value)
  {
    this.Value = value;
  }

  public override string ToString()
  {
    return this.Value.ToString();
  }

  public DateTime Value
  {
    get;
    private set;
  }
}
```

Nope:

```
Error 1 Structs cannot contain explicit parameterless constructors

Error 2 The 'this' object cannot be used before all of its fields are assigned to

Error 3 The 'this' object cannot be used before all of its fields are assigned to 
```

OK ... well, I'm out of ideas. It doesn't seem like automated properties play nicely with value types, unless I'm missing something (which is probably the case - I very rarely make a struct these days and I might be forgetting the obvious solution).

> Published: 11.05.2008 02:54:40 PM CST