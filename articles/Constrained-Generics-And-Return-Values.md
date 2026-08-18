---
title: Constrained Generics and Return Values
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Constrained Generics and Return Values

I was working on some mocking stuff today and I ran into an odd situation. Let's say you have the following class hierarchy:

```c#
public abstract class Cup
{
}

public class CoffeeCup : Cup
{
}

public class Shotglass : Cup
{
}
```

Now, let's define a factory that randomly creates cups:

```c#
public static class CupCreator
{
  public static T Create<T>() where T : Cup
  {
    if(DateTime.Now.Second % 2 == 0)
    {
      return new Shotglass();
    }
    else
    {
      return new CoffeeCup();
    }
  }
}
```

The problem is, the two return statements are in error - here's what the error statement looks like:

```
Cannot implicitly convert type 'Shotglass' to 'T'
```

It's error [CS0029](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs0029).

Here's the funny thing. 10 years ago, I would've assumed that there's something wrong with the compiler. I'm older now and I know that I'm probably the idiot :).

When I first did this, I thought, OK, `T` must be at least a `Cup` ... so returning a `CoffeeCup` or a `Shotglass` should be OK, right?

Well, no. What if I call it like this?

```c#
var shot = CupCreator.Create<Shotglass>();
```

Then returning a `CoffeeCup` is invalid.

Just because `T` is constrained to be a `Cup` and the two return types descend from `Cup` doesn't mean the code is valid. It took me a bit to figure this one out!

> Published: 08.18.2008 02:25:43 PM CST