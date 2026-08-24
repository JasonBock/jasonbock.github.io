---
title: Stupid Special .NET Types
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Stupid Special .NET Types

For the past 6 months, I haven't done as much techincal blogging as I have in the past. I don't apologize for that - I'm just stating a fact. To be more specific, my custom blogging tool doesn't work at the client I'm at, and my temporary workaround (a secure web page) isn't cutting it. Therefore, I usually wait to blog until I get home, but now that I'm a father I don't feel like geeking out when I get home as much as I used to, nor am I always motivated to blog about things I've run into at work. I plan on adding web services to make my custom blogging engine more accessible ... someday in the future.

Anyway, I ran into an issue yesterday that kind of annoyed me a bit. It's not that big of a deal, but ... well, here's the issue. If you've used .NET 2.0, you may have noticed that there's a static method on the string class called `IsNullOrEmpty()`:

```c#
string nullString = null;
string emptyString = string.Empty;
string name = "Jason Bock";

Console.Out.WriteLine(string.IsNullOrEmpty(nullString));
Console.Out.WriteLine(string.IsNullOrEmpty(emptyString));
Console.Out.WriteLine(string.IsNullOrEmpty(name));
```

It's nice because there are a lot of situations I run into where a string being null or empty really means the same thing to me, and this utility method takes care of the following code that I always have to write in .NET 1.x:

```c#
if(nullString != null && nullString.Length > 0)
```

As I was working with some code that uses a bunch of arrays of different types, I realized that it would be cool if the `Array` class had a similar static method called `IsNullOrEmpty()` so I could do something like this:

```c#
string[] nullNames = null;

Console.Out.WriteLine(Array<string[]>.IsNullOrEmpty(nullNames));
```

Unfortunately, such a method doesn't exist. I tried writing my own, but my first attempt came to crashing end:

```c#
bool IsNullOrEmpty<T>(T array) where T : Array
{
  // ...
}
```

The problem with this is the generic constraint. I can't use array - the compiler gives the following error: "Constraint cannot be special class 'System.Array'". Mike Woodring recently ran into the same problem (although in a different context and with the `Delegate` class). This sucks, but I started wondering if the `Array` class was updated in 2.0 from 1.x. In 1.x, `Array` implemented the `ICloneable`, `IList`, `ICollection`, and `IEnumerable` interfaces. In 2.0, `Array` also implements the generic versions of these interfaces at runtime, but the key point is that `Array` implements IList, so now we can make some headway:

```c#
static bool IsNullOrEmpty<T>(T value) where T : IList
{
  return !(value != null && value.Count > 0);
}
```

Now I can write code like this:

```c#
string[] nullNames = null;
string[] emptyNames = new string[0];
string[] names = { "Jason", "Liz" };

Console.Out.WriteLine(Program.IsNullOrEmpty<string[]>(nullNames));
Console.Out.WriteLine(Program.IsNullOrEmpty<string[]>(emptyNames));
Console.Out.WriteLine(Program.IsNullOrEmpty<string[]>(names));

int[] nullAges = null;
int[] emptyAges = new int[0];
int[] ages = { 34, 34 };

Console.Out.WriteLine(Program.IsNullOrEmpty<int[]>(nullAges));
Console.Out.WriteLine(Program.IsNullOrEmpty<int[]>(emptyAges));
Console.Out.WriteLine(Program.IsNullOrEmpty<int[]>(ages));
```

One minor issue is that `IsNullOrEmpty()` is not limited to arrays. For example, the following code will work as well:

```c#
ArrayList identifiers = new ArrayList(
  new Guid[] { Guid.NewGuid(), Guid.NewGuid() });

Console.Out.WriteLine(Program.IsNullOrEmpty<ArrayList>(identifiers));
```

I can change `IsNullOrEmpty()` to enforce that the given value is an array and only an array:

```c#
static bool IsNullOrEmpty<T>(T value) where T : IList
{
  if(!typeof(T).IsArray)
  {
    throw new ArgumentException("value is not an array.", "value");
  }

  return !(value != null && value.Count > 0);
}
```

However, I'm not to thrilled about this. I mean, since I can't constrain the generic value to an `Array`, then why enforce it? The ideal solution would be to constrain the generic type to an array, but that's not possible, so I might as well let any type in that implements `IList`.

Generics are fun. I love 'em. But sometimes there are certain situations where you're still limited by "interesting" edge cases in .NET. In this case, though, it doesn't prevent me from creating a reasonable implementation to solve the problem.

> Published: 12.14.2005 09:20:42 PM CST