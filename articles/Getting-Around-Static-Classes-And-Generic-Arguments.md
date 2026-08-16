---
title: Getting Around Static Classes and Generic Arguments
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Getting Around Static Classes and Generic Arguments

A while ago I wrote about C#'s restriction with static classes being used for generic arguments. Yesterday I ran into this scenario again for `ScopeSwitcher` in Spackle.NET, and I thought of a way to get around it:

```c#
private static void Switch<TNew>(Type target, Action code, params object[] args)
{
  IDisposable switcher = null;

  try
  {
    switcher = Activator.CreateInstance(
      typeof(ScopeSwitcher<,>).MakeGenericType(
        target, typeof(TNew)), args) as IDisposable;
    code();
  }
  finally
  {
    if(switcher != null)
    {
      switcher.Dispose();
    }
  }
}
```

Granted, this isn't a very clean approach. But it does give me the ability to write code like this:

```c#
[TestMethod]
public void SwitchOnStaticClass()
{
  var currentValue = Guid.NewGuid();
  var newValue = Guid.NewGuid();

  StaticValuePropertyOnStaticClass.Id = currentValue;
  ScopeSwitcherHelpers.Switch<Guid>(typeof(StaticValuePropertyOnStaticClass), newValue, () =>
  {
    Assert.AreEqual(newValue, StaticValuePropertyOnStaticClass.Id);
  });

  Assert.AreEqual(currentValue, StaticValuePropertyOnStaticClass.Id);
}
```

Note that `StaticValuePropertyOnStaticClass` is a `static` class.

Now, in my case, I only need the variable switcher around for a short time. If I needed to use that type for a number of interactions, I'd probably write a `DynamicMethod` to improve performance. But at least there is a way in C# to use a static class as a generic argument: just use `MakeGenericType()`.

> Published: 11.18.2008 07:15:28 AM CST