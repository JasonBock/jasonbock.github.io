---
title: Let It Fail Naturally
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Let It Fail Naturally

Yesterday after I worked on my `Shuffle()` extension method for [Spackle](https://www.nuget.org/packages/Spackle), something started rolling around in my head. It was this code that I saw in the same file:

```c#
public static void Swap<T>(this IList<T> @this, int x, int y)
{
  @this.CheckParameterForNull("@this");
  IListOfTExtensions.CheckIndex(@this, x, "x");
  IListOfTExtensions.CheckIndex(@this, y, "y");

  if(x != y)
  {
    T xValue = @this[x];
    @this[x] = @this[y];
    @this[y] = xValue;
  }
}
```

`CheckIndex()` looks at the given index and throws an `ArgumentException` if it's outside the boundaries.

This was starting to bother me. Why?

Because the exceptional condition of accessing a list beyond its valid index boundaries is already handled by `List<T>`.

But wait! I'm actually taking an `IList<T>` reference, not a `List<T>`. So there's no guarantee that the implementor will actually throw the exception.

But...I'm OK with removing the code:

```c#
public static void Swap<T>(this IList<T> @this, int x, int y)
{
  @this.CheckParameterForNull("@this");

  if(x != y)
  {
    T xValue = @this[x];
    @this[x] = @this[y];
    @this[y] = xValue;
  }
}
```

Most of the time, the underlying implementation will be a `List<T>`. And even if it's not, I'm guessing that the implementor will check the index as the docs state that it should throw `ArgumentOutOfRangeException` (or `NotSupportedException` if the list is read-only).

Of course, with Code Contracts in 4.0, you can add contract specifications to interfaces. But we're not quite there yet :)

The moral of the story is, sometimes you can be overly defensive. There's no right or wrong answer for every case, but take the time to review your code to ensure it behaves the way the user thinks it should. In this case, having an exception thrown that's stated in the documentation makes a lot more sense.

> Published: 08.21.2009 08:16:41 AM CST