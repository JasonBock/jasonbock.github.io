---
title: Why So Many Overloads for StringBuilder's Append?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Why So Many Overloads for StringBuilder's Append?

Recenty I've been trying to figure out how to turn a `TypeReference` into a `Type` in [Cecil](https://github.com/jbevain/cecil). The reason is that I'm taking a bunch of values and putting them together via a `StringBuilder`, like this:

```c#
var random = new Random();
var id = Guid.NewGuid();
var one = 1;
var data = "data";

var builder = new StringBuilder();
builder.Append(one).Append(data)
  .Append(random).Append(id);
```

But trying to find the version of `Append()` that most closely matches a particular type in Cecil isn't easy:

```c#
var append = @this.Module.Import(typeof(StringBuilder)
  .GetMethod("Append", 
    new Type[] { ConvertToType(property.PropertyType) })); 
```

In that code, I have a `TypeReference` via `PropertyType`, and I want to find the `Append()` method via a call to `GetMethod()`, which requires a `Type`. `ConvertToType()` is a mythical method that I don't have yet, which would convert the `TypeReference` to a Type.

Now, someone on a thread made a comment basically asking "why would I want to do this?", pointing to the fact that most of the `Append()` methods just end up calling `ToString()` anyway. So he's basically saying, why don't I emit code like this:

```c#
var random = new Random();
var id = Guid.NewGuid();
var one = 1;
var data = "data";

var builder = new StringBuilder();
  builder.Append(one.ToString()).Append(data)
    .Append(random.ToString()).Append(id.ToString());
```

I haven't tried to emit this with Cecil yet, but I'm thinking this might be easier. I do have to do some boxing of the value types to call `ToString()` on them correctly, but that shouldn't be too hard. There's also another option: cast everything to object and just call that one version of `Append()`:

```c#
var random = (object)(new Random());
var id = (object)Guid.NewGuid();
var one = (object)1;
var data = (object)"data";

for(var i = 0; i < Program.Iterations; i++)
{
  var builder = new StringBuilder();
  builder.Append(one).Append(data)
    .Append(random).Append(id);
}
```

I was curious, though...is there a performance difference between these approaches?

```
SpecificTypes: 9.284
ToString: 9.506
AllObjects: 9.423
```

These are the results of running each approach 90 million times, and then running the test 3 times. Basically ... I don't see a huge difference between any of them, so I'm going to explore calling `ToString()` on them as that may be the best way to do this in Cecil. From a language perspective, I definitely like the first way as it looks much cleaner to me, but I'm emitting code in this case so aesthetics isn't really important!

> Published: 11.11.2008 07:47:29 AM CST