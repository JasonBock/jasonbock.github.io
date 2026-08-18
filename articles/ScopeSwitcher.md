---
title: ScopeSwitcher
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# ScopeSwitcher

UPDATE (8/6/2008, 8:39 PM): Based on some comments, I've changed the implementation slightly to use `FirstOrDefault()`.

One of the first things I figured out when I got into .NET was how to switch the cursor to the hourglass icon. I know, that's not a very earth-shattering thing to do, but it's something I always did in VB, so I wanted to know how to do it. As it turned out, it's not that hard:

```c#
Cursor.Current = Cursors.WaitCursor;
```

Of course, I wanted to make sure that the cursor went back to the original value once I was done with my code, so I wrote this:

```c#
var current = Cursor.Current;
Cursor.Current = Cursors.WaitCursor;

try
{
  // Do interesting code here...
}
finally
{
  Cursor.Current = current;
}
```

If you're OK that you can "misuse" `IDisposable`, then you can create a little utility class:

```c#
public sealed class CursorSwitcher : IDisposable
{
  private Cursor current = Cursor.Current;
 
  public CursorSwitcher(Cursor newValue) 
    : base()
  {
    Cursor.Current = newValue;
  }
 
  public void Dispose()
  {
    Cursor.Current = this.current;
  }
}
```

This packages up the cursor switching nicely:

```c#
using(var cursorSwitcher = new CursorSwitcher(Cursors.WaitCursor))
{
  // Do interesting code here...
}
```

However, changing the cursor value isn't the only time I've wanted to switch out a value. There have been numerous situations where I've wanted to swap a property value with another one for a specific section of code, and then switch it back. So, let's generalize the pattern:

```c#
public sealed class ScopeSwitcher<TTarget, TNew> : IDisposable
{
  private bool isPropertyStatic;
  private TNew newValue;
  private TNew oldValue;
  private PropertyInfo property;
  private TTarget target;

  public ScopeSwitcher(TNew newValue)
    : base()
  {
    this.FindProperty(BindingFlags.Static);
    this.isPropertyStatic = true;
    this.ExchangePropertyValue(newValue);
  }

  public ScopeSwitcher(TNew newValue, string propertyName)
    : base()
  {
    if(string.IsNullOrEmpty(propertyName))
    {
      throw new ArgumentException("No property name was given.", "propertyName");
    }

    this.FindProperty(BindingFlags.Static, propertyName);
    this.isPropertyStatic = true;
    this.ExchangePropertyValue(newValue);
  }
    
  public ScopeSwitcher(TTarget target, TNew newValue)
    : base()
  {
    if(target == null)
    {
      throw new ArgumentNullException("target");
    }
        
    this.FindProperty(BindingFlags.Instance);
    this.target = target;
    this.ExchangePropertyValue(newValue);
  }

  public ScopeSwitcher(TTarget target, TNew newValue, string propertyName)
    : base()
  {
    if(target == null)
    {
      throw new ArgumentNullException("target");
    }

    if(string.IsNullOrEmpty(propertyName))
    {
      throw new ArgumentException("No property name was given.", "propertyName");
    }
        
    this.FindProperty(BindingFlags.Instance, propertyName);
    this.target = target;
    this.ExchangePropertyValue(newValue);
  }

  private void ExchangePropertyValue(TNew newValue)
  {
    this.newValue = newValue;
        
    if(this.isPropertyStatic)
    {
      this.oldValue = (TNew)this.property.GetValue(null, null);
      this.property.SetValue(null, this.newValue, null);                    
    }
    else
    {
      this.oldValue = (TNew)this.property.GetValue(this.target, null);
      this.property.SetValue(this.target, this.newValue, null);                                
    }
  }
    
  private void FindProperty(BindingFlags flags)
  {
    this.property = (from property in typeof(TTarget).GetProperties(
      BindingFlags.Public | flags)
      where property.PropertyType.IsAssignableFrom(typeof(TNew))
      where property.CanRead && 
        property.GetGetMethod() != null && 
        property.GetGetMethod().IsPublic
      where property.CanWrite && 
        property.GetSetMethod() != null && 
        property.GetSetMethod().IsPublic
      select property).FirstOrDefault();

    if(this.property == null)
    {
      throw new PropertyNotFoundException(
        "A read/write property was not found that was assignable from " + 
        typeof(TNew).FullName + ".");
    }
  }

  private void FindProperty(BindingFlags flags, string propertyName)
  {
    this.property = (from property in typeof(TTarget).GetProperties(
      BindingFlags.Public | flags)
      where property.Name == propertyName
      where property.PropertyType.IsAssignableFrom(typeof(TNew))
      where property.CanRead && 
        property.GetGetMethod() != null && 
        property.GetGetMethod().IsPublic
      where property.CanWrite && 
        property.GetSetMethod() != null && 
        property.GetSetMethod().IsPublic
      select property).FirstOrDefault();

    if(this.property == null)
    {
      throw new PropertyNotFoundException(
        "A read/write property with name " + propertyName +
        " was not found that was assignable from " + typeof(TNew).FullName + ".");
    }
  }

  public void Dispose()
  {
    if(this.property != null)
    {
      if(this.isPropertyStatic)
      {
        this.property.SetValue(null, this.oldValue, null);
      }
      else
      {
        this.property.SetValue(this.target, this.oldValue, null);
      }
    }
  }
}
```

That's a lot of code to digest (you can get all of the code plus tests here) but what's nice is that you can do something like this:

```c#
using(var switcher = new ScopeSwitcher<Cursor, Cursor>(Cursors.WaitCursor))
{
  // Do interesting code here...
}
```

Or, if you had a class like this:

```c#
public sealed class StringData
{
  private StringData()
    : base()
  {
  }
 
  public static string Data
  {
    get;
    set;
  }
}
```

You can use the same class:

```c#
using(var switcher = new ScopeSwitcher<StringData, string>("new data."))
{
  // Do interesting code here...
}
```

If the class has a number of properties that match the value of `TNew`, you can specify the name of the property to switch. So, if `StringData` was updated like this:

```c#
public sealed class StringData
{
  private StringData()
    : base()
  {
  }
 
  public static string Data
  {
    get;
    set;
  }

  public static string Value
  {
    get;
    set;
  }
}
```

You can choose which property to change:

```c#
using(var switcher = new ScopeSwitcher<StringData, string>("new data.", "Value"))
{
  // Do interesting code here...
}
```

This also works for instance values and not just static properties.

Let me know if you have any issues with it - enjoy!

> Published: 08.01.2008 01:51:05 PM CST