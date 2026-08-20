---
title: Assert.AreEqual() Gotcha
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `Assert.AreEqual()` Gotcha

Sometimes as I'm writing a test I'll accidentally write code like this:

```c#
Assert.AreEqual(3, myList);
```

My intention was to actually check the number of items in the collection - IOW the test should have been written like this:

```c#
Assert.AreEqual(3, myList.Count);
```

But since the signature of `AreEqual()` looks like this:

```c#
public static void AreEqual(object expected, object actual);
```

there's no type checking. It doesn't matter that `AreEqual()` is overloaded to the hills with "core" types and has a generic verison, because when I write code where the expected and actual values are not the same type, the compiler will always default to the most generic one it can find, and in this case it's the one I just mentioned above.

I also find writing code like this kind of annoying:

```c#
Assert.AreEqual<int>(3, myList.Count);
```

It makes it clear what my intented type to test is, but it just feels tedious, especially if I have to specify T every time I call `AreEqual()`.

Thinking out loud [1], I'm wondering if `Assert` should have one and only one `AreEqual()` method, the generic one:

```c#
public static void AreEqual<T>(T expected, T actual);
```

OK, you'd still need the message overload:

```c#
public static void AreEqual<T>(T expected, T actual, string message);
```

But I think you could get away with removing a lot of overloads. If you want to compare two object types (which I've found I rarely, if ever, do), you can still do that:

```c#
Assert.AreEqual<object>(3, myList);
```

If you want to check two strings for equality using case-insensitive comparison and a specific `CultureInfo` value, well, shoot, just use `Compare()` on `String`:

```c#
Assert.AreEqual(0, string.Compare("this", "This", true, new CultureInfo("en-US")));
```

If you want to check two numeric values with a delta parameter ... well, OK, maybe you'd have to leave in the overloads that deal with the numeric types (like `float`).

I realize that `Assert` has all the overloads on it to make it easier to test certain types. But I've found that it also can lead to writing code that compiles but the tests fail because of a stupid coding error. I should try using my own `CustomAssert` class that only exposes what I want, something like this:

```c#
public static class CustomAssert
{
  public static AreEqual<T>(T expected, T actual)
  {
    Assert.AreEqual<T>(expected, actual);
  }
}
```

This is incomplete, of course, but you get the idea. Then I'd use `CustomAssert` instead of `Assert`.

[1] Which means I may look stupid afterwards because of it :)

> Published: 12.10.2007 08:21:26 AM CST