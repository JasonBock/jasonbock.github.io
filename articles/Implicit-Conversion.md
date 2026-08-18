---
title: Implicit Conversion
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Implicit Conversion

Recently I ran into a problem where I had to dive into the C# bag of tricks. It's not a complicated trick and it's been around for quite some time (which is why I had to dive into the bag, rathen than skim the LINQ surface). But I've never used it until now, so here's a quick review of implicit conversions.

I needed to create a percentage value. Basically, I wanted to restrict a `decimal` between the values of 0 to 100 inclusive [1]. So rather than spread that rule around in code, I created a tiny struct:

```c#
using System;

namespace ImplicitConversion
{
  public struct Percentage
  {
    private decimal value;

    public Percentage(decimal value)
    {
      if(value < 0m || value > 100m)
      {
        throw new ArgumentException("The value must be between 0 and 100 inclusive.", "value");
      }

      this.value = value;
    }

    public decimal Value
    {
      get
      {
        return this.value;
      }
    }
  }
}
```

OK, that works [2], but then I created a function like this:

```c#
static void Report(Percentage one, Percentage two)
{
  Console.Out.WriteLine(one);
  Console.Out.WriteLine(two);
}
```

and I accidentally used it like this:

```c#
Program.Report(10m, 20m);
```

Of course, to fix it, I could've done this:

```c#
Program.Report(new Percentage(10m), 
  new Percentage(20m));
```

but that felt ... unnatural. Why couldn't I just convert it? That would be so cool if I could...hey, wait a minute! C# has implicit conversion:

```c#
using System;

namespace ImplicitConversion
{
  public struct Percentage
  {
    private decimal value;

    public Percentage(decimal value)
    {
      if(value < 0m || value > 100m)
      {
        throw new ArgumentException("The value must be between 0 and 100 inclusive.", "value");
      }

      this.value = value;
    }

    public static implicit operator decimal(Percentage value)
    {
      return value.value;
    }

    public static implicit operator Percentage(decimal value)
    {
      return new Percentage(value);
    }

    public decimal Value
    {
      get
      {
        return this.value;
      }
    }
  }
}
```

And all was well again.

Again, I've never used implict (or explicit) conversion since C# came out. But in this case it seems like a natural fit.

[1] Yes, I realize a percentage can be negative and go beyond the upper range I gave. Just ignore that for now.

[2] I would've like to make it generic, but then I would've wanted to constrain the type to unsigned numerics, and that's not doable in the `where` clauses. Plus, I just needed to get this done with the `decimal` type, so I declared YAGNI :).

> Published: 05.12.2008 01:13:20 PM CST