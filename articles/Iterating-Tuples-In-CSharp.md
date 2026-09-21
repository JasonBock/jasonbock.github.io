---
title: Iterating Tuples in C#
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Iterating Tuples in C#

## Inspired by Python

Recently I was at [VSLive](https://vslive.com/) San Diego, and I had some time to watch a couple of presentations. Usually when I'm speaking at a conference, I'm also working, so as soon as I'm done being on stage I'm back at my work laptop. This time, I was on vacation, so I decided to watch [Ted Neward](https://www.newardassociates.com/)'s "[Busy Developer's Guide to Python](https://www.newardassociates.com/presentations/BusyDevsGuide/Python.html)". I have a basic understanding of how Python works, but there were a couple of features Ted mentioned that I didn't know about. Specifically, that you can [iterate tuples](http://slides.newardassociates.com/BusyDevsGuide/Python.html#(82)):

```python
my_first_tuple = "Ted", "Neward", 47
for t in my_first_tuple:
    print(t)
```

> If you're really curious, you can find out more about tuples in Python by [looking at the docs](https://docs.python.org/3/builtins/stdtypes.html#tuples).
> 
> Also, I pulled that code from Ted's slide. Note the URL has a number in it, so if he updates the content, the indexing may be off, but you should be able to find the example fairly quickly - it's in the *Flow Control* section.
> 
> Also note that the example is hinting that Ted is 47 years young, and that is clearly a lie.

That got me thinking ...

Could you iterate tuples in C#?

## A Quick Overview of Tuples

Before I get too far, let's cover what a tuple is and how it works in C#. [The docs](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/tuples) have all the information you need on tuples, but let's start with a simple example:

```c#
var person = (Guid.NewGuid(), "Joe Smith", 25);
```

In this case, `person` is a `ValueTuple<Guid, string, int>`. There are a number of `ValueTuple` types defined in C# to handle as large of a tuple that you want, though typically you'd probably make tuples with only a couple of values in them. One of the main use cases for tuples is to make it easy to return multiple values from a method:

```c#
var (id, name, age) = service.GetPerson();
```

In that contrived example, a service is used to retrieve person information via `GetPerson()`. Note that you can also name the values contained within in the tuple if you want. If you don't, a tuple always has `Item` properties that positionally relate to each tuple item:

```c#
var person = service.GetPerson();
Console.WriteLine(person.Item2);
```

In this case, you'll see the name of the person in the console window.

By the way, don't use tuples if you want the content of the type to have "meaning". If you really want to define a `Person` that's used in a number of places within your application, then create a type that represents a person - a record is a convenient way of doing just that:

```c#
public record Person(Guid Id, string Name, uint Age);
```

## Iterating Tuples

OK, so that's the essentials of tuples. Now, to get back to the question of, can I iterate tuples?

```c#
var person = (Guid.NewGuid(), "Joe Smith", 25);

foreach (var item in person)
{
  Console.WriteLine(item);
}
```

Turns out, you can't:

```
CS1579: foreach statement cannot operate on variables of type '(Guid, int, string)' because '(Guid, int, string)' does not contain a public instance or extension definition for 'GetEnumerator'
```

The problem is that tuples are not enumerable. If you look at [the definitions of all eight `ValueTuple` types](https://source.dot.net/#System.Private.CoreLib/src/runtime/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs) in .NET, none of them derive from `IEnumerable<>`, or `IEnumerable`, or define a method called `GetEnumerator()` that returns `IEnumerator`. It makes sense that `foreach` can't work on tuples.

But ... can we make it work?

As Bob the Builder would say, "yes we can!"

Note that I said that one of the ways an object can be enumerated is if the type defines a `GetEnumerator()` method. It doesn't have to implement `IEnumerable<>`, though it should if you can do it - that's the preferred way to support enumeration of a type. In fact, if you read the error message for [`CS1579`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/foreach-diagnostics) carefully, it says at the end: `or extension definition for 'GetEnumerator'`. We don't own the tuple types as they're defined in .NET itself, but we can create extensions methods for them:

```c#
static class ValueTupleExtensions
{
  extension<T1, T2, T3>(ValueTuple<T1, T2, T3> tuple)
  {
    public IEnumerator GetEnumerator()
    {
      yield return tuple.Item1;
      yield return tuple.Item2;
      yield return tuple.Item3;
    }
  }
}
```

By extending `ValueTuple<T1, T2, T3>` with a `GetEnumerator()` method that returns `IEnumerator`, we can use `yield return` on the three items. Then everything works!

## One Extension Method to Handle All Tuples

However, this isn't complete. I've just created **one** extension method for the tuple that has three items. As I mentioned before, there are eight `ValueTuple` types, so we'd have to create eight `GetEnumerator()` extension methods. To complicate things further, `ValueTuple<T1, T2, T3, T4, T5, T6, T7, TRest>` is special in that the 8th generic value can either be a value, or **another** tuple. This is the extensibility point where tuples can house any number of values, and the compiler handles that by using the 8-item `ValueTuple` as many times as it needs to by populating `TRest` with another `ValueTuple`. That can be recursive as well - meaning, a tuple with 16 values like this:

```c#
var lotsOfItems = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16);
```

is effectively composed of 3 `ValueTuple`s: 2 8-value `ValueTuple` types that each hold 7 values, and a 2-value `ValueTuple` that holds the last 2 values. The definition is this: `ValueTuple<int, int, int, int, int, int, int, ValueTuple<int, int, int, int, int, int, int, ValueTuple<int, int>>>`. That's not the easiest type definition to visually parse, but it's what `lotsOfItems` is.

This would complicate things with our 8-value `GetEnumerator()` extension. We'd have to check to see if `Item8` is **another** `ValueTuple`, and if it is, cast it correctly and enumerate that tuple. It wouldn't be too difficult, but it turns out there's a much easier way to handle tuple iteration with **one** extension method.

If you dig into the definition of all the `ValueTuple` types, they all implement an interface called [`ITuple`](https://source.dot.net/#System.Private.CoreLib/src/runtime/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/ITuple.cs). Here's what `ITuple` looks like:

```c#
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.

namespace System.Runtime.CompilerServices
{
  /// <summary>
  /// This interface is required for types that want to be indexed into by dynamic patterns.
  /// </summary>
  public interface ITuple
  {
    /// <summary>
    /// The number of positions in this data structure.
    /// </summary>
    int Length { get; }

    /// <summary>
    /// Get the element at position <param name="index"/>.
    /// </summary>
    object? this[int index] { get; }
  }
}
```

This is perfect for our needs, because we can get the number of items in any tuple via the `Length` property, and then `yield return` each item through the indexer:

```c#
static class ValueTupleExtensions
{
  extension(ITuple tuple)
  {
    public IEnumerator GetEnumerator()
    {
      for (var i = 0; i < tuple.Length; i++)
      {
        yield return tuple[i];
      }
    }
  }
}
```

Now, no matter what `ValueTuple` is used, we can easily iterate its' values:

```c#
var person = (Guid.NewGuid(), "Joe Smith", 25);

foreach (var item in person)
{
  Console.WriteLine(item);
}
```

You'll now see something like this in the console window:

```
1353ae66-638b-47ea-8d44-13f685c9869a
42
Jason
```

## Why Iterate Tuples at All?

OK, so this all works. While tuples are not iterable out of the box, it's simple to add that support with a small extension method. I started to wonder, though, why this capability isn't provided for developers in .NET proper. Maybe it was never part of the tuple specification. Maybe it was a considered feature, but ultimately dropped. I found [this proposed issue](https://github.com/dotnet/runtime/issues/20346) that references [this discussion](https://github.com/dotnet/csharplang/discussions/600), which leads to a **lot** more links that I'll let you dive into if you're interested in discovering more (I guess I'm kind of late to the game with respect to the discussion of enumerating tuples!). The fact of the matter is that tuples don't have defined enumeration support, and it may never be added. As Stephen Toub said in the issue link in the previous paragraph:

> "anyone can write the simple extension methods to do this themselves and even publish it to NuGet for others to consume"

For what it's worth, I've decided to add this extension method to my [Spackle](https://github.com/JasonBock/SpackleNet) library for the [`14.1.0` release](https://www.nuget.org/packages/Spackle/14.1.0). If you want to just use a NuGet package that has this extension, feel free to reference it.

While I was adding this extension method to Spackle, I started to wonder **why** anyone would want to iterate a tuple's value. The only reason I think of was for debugging - a simple way to log a tuple's contents without regard to the number of tuple elements - but even that's a bit of a stretch. I considered that maybe it might be a novel way to store data of arbitrary types within a collection. For example, instead of doing this:

```c#
List<object> items =
[
  Guid.NewGuid(), Guid.NewGuid().ToString(), RandomNumberGenerator.Next(), RandomNumberGenerator.NextDouble()
];

foreach (var item in items)
{
  // Do something with item ...
}
```

You could do this:

```c#
var items =  
(
  Guid.NewGuid(), Guid.NewGuid().ToString(), RandomNumberGenerator.Next(), RandomNumberGenerator.NextDouble()
);

foreach (var item in items)
{
  // Do something with item ...
}
```

Needless to say, as soon as I thought of this, I wanted to see how it would perform. I added `TupleVsListEnumeration` to the `Spackle.Performance` library to compare these two approaches. I changed the initial size of the values within the list and the tuple and grabbed the results ... which speak for themselves:

*4 Elements*

| Method         | Mean      | Error     | StdDev    | Ratio | RatioSD | Gen0   | Allocated | Alloc Ratio |
|--------------- |----------:|----------:|----------:|------:|--------:|-------:|----------:|------------:|
| EnumerateList  |  1.623 ns | 0.0097 ns | 0.0091 ns |  1.00 |    0.01 |      - |         - |          NA |
| EnumerateTuple | 16.221 ns | 0.0560 ns | 0.0496 ns |  9.99 |    0.06 | 0.0069 |     120 B |          NA |

*8 Elements*

| Method         | Mean      | Error     | StdDev    | Ratio | RatioSD | Gen0   | Allocated | Alloc Ratio |
|--------------- |----------:|----------:|----------:|------:|--------:|-------:|----------:|------------:|
| EnumerateList  |  2.085 ns | 0.0211 ns | 0.0187 ns |  1.00 |    0.01 |      - |         - |          NA |
| EnumerateTuple | 30.874 ns | 0.1125 ns | 0.1052 ns | 14.81 |    0.14 | 0.0116 |     200 B |          NA |

*16 Elements*

| Method         | Mean      | Error     | StdDev    | Ratio | RatioSD | Gen0   | Allocated | Alloc Ratio |
|--------------- |----------:|----------:|----------:|------:|--------:|-------:|----------:|------------:|
| EnumerateList  |  4.713 ns | 0.0299 ns | 0.0280 ns |  1.00 |    0.01 |      - |         - |          NA |
| EnumerateTuple | 73.828 ns | 0.4033 ns | 0.3773 ns | 15.66 |    0.12 | 0.0209 |     360 B |          NA |

*32 Elements*

| Method         | Mean       | Error     | StdDev    | Ratio | RatioSD | Gen0   | Allocated | Alloc Ratio |
|--------------- |-----------:|----------:|----------:|------:|--------:|-------:|----------:|------------:|
| EnumerateList  |   7.871 ns | 0.0179 ns | 0.0149 ns |  1.00 |    0.00 |      - |         - |          NA |
| EnumerateTuple | 447.998 ns | 1.0910 ns | 0.9672 ns | 56.91 |    0.16 | 0.0391 |     680 B |          NA |

*64 Elements*

| Method         | Mean        | Error    | StdDev   | Ratio  | RatioSD | Gen0   | Allocated | Alloc Ratio |
|--------------- |------------:|---------:|---------:|-------:|--------:|-------:|----------:|------------:|
| EnumerateList  |    20.72 ns | 0.145 ns | 0.136 ns |   1.00 |    0.01 |      - |         - |          NA |
| EnumerateTuple | 3,553.08 ns | 5.496 ns | 4.589 ns | 171.45 |    1.10 | 0.0763 |    1320 B |          NA |

I went farther than this, but you can easily see the trend. Basically, don't use a tuple to store a bunch of objects as a "collection". A `List<object>` is better!

> If you want to get a better understanding as to why the performance of iterating a tuple stinks relative to iterating a `List<object>`, look at the implementations of the `ValueTuple` types. Specifically, look at the indexers. Implementing the `Length` property isn't too bad, even when you have a tuple with more than 8 elements. The indexer needs to map each index value with the appropriate `Item` property, and it gets worse when you have large tuples that require special handling of the `TRest`-based property.

## Summary

There's a lot of value in looking at other programming languages, even if you don't use them on a day-to-day basis. In all likelihood, they'll have features that your language(s) of choice don't have, and it can inspire you to dig deeper into the language(s) you use.

> Published: 09.21.2026 11:09:41 PM CST