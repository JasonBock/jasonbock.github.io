---
title: What's In a Hash Code?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# What's In a Hash Code?

## Abstract

.NET developers have wrestled with creating hash codes for objects ever since .NET was released. In this article, I'll cover modern approaches to hash code calculation and discuss the differences between them. You'll learn the fundamentals and rules for hash codes and how creating hash codes has become simpler in recent history.

## The Importance Of Hash Codes

Hash codes usually don't get a lot of discussion among developers. We're too busy discussing the latest UI framework, learning to add resiliency into our distributed applications or wondering where the error is in our YAML file. But while they're not always high on our radar, there must be a reason hash codes are a core part of the .NET type system and available for every .NET application.

We only use hash codes for one thing: putting an object in a hash table. A common misconception is that hash codes are used to determine if two objects are equal. While implementing object equality requires a developer to provide an implementation for `GetHashCode()`, the hash code itself doesn't determine equality. Hash codes are a 32-bit value used by types that use hash tables, such as a `HashSet<T>`. For example, let's say we had a class that defined a person via their age and name. It would make sense that these two objects would be equal, as shown in Listing 1.

*Listing 1 - Comparing Two Objects For Equality*

```c#
var person1 = new Person(33, "Jane");
var person2 = new Person(33, "Jane");
Console.Out.WriteLine(person1 == person2);
```

When we run the code in Listing 1, we'd expect to see `true` in the console window. We would expect that same result from the code in Listing 2.

*Listing 2 - Comparing Hash Code Values*

```c#
Console.Out.WriteLine(
  person1.GetHashCode() == person2.GetHashCode());
```

However, just because two objects have the same hash code (and they're also the same type) doesn't mean they're equal. `GetHashCode()` has more than 4 billion unique values, so at some point you're going to run into two objects that have the same hash code, but they're not equal. That said, for data types that use hash tables, you can significantly improve performance with lookups if the hash code values are unique enough for a reasonable set of objects.

What does "unique enough" mean? There's no hard-and-fast rule here, but the more objects you add to a collection that uses hash tables, the greater the chance you'll run into a collision. If you're storing more than a million objects in a `HashSet<T>` object, you should probably go back to the drawing board anyway (or at least look at the memory pressure you're incurring and know the ramifications).

So, how does `HashSet<T>` work with hash codes? Let's look at an example in Listing 3.

*Listing 3 - Hash Codes And HashSet*

```c#
var people = new HashSet<Person>();

var person1 = new Person(22, "Jeffrey");
people.Add(person1);

var person2 = new Person(22, "Jeffrey");
Console.Out.WriteLine(people.Contains(person2));
```

The code in Listing 3 will print out `false` even though `GetHashCode()` for `person1` and `person2` would be the same. The implementation of [`HashSet<T>`](https://source.dot.net/#System.Private.CoreLib/src/runtime/src/libraries/System.Private.CoreLib/src/System/Collections/Generic/HashSet.cs) uses "buckets" and "entries" to make lookups much faster than traversing the entire list. The details are rather complex, but focus on the `Contains()` method. You'll see that the hash code for the given item is used to find an index value in its buckets field, which is used to find a specific entry in its entries field. `Contains()` will still do an equality comparison and not rely on the hash code itself. Again, hash codes help with lookup speed. They do not determine object equality.

## Rules For Implementing `GetHashCode()`

At first glance, `GetHashCode()` looks like a simple method to implement. It has no parameters and it returns an `int`. Can't get any easier than that, right? Not so fast - looks are deceiving.

You must adhere to several rules and conventions when implementing this method. You can find the details on these (and more!) in the Microsoft .NET documentation for ["Object.GetHashCode Method"](https://learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode) and ["Guidelines and Rules for GetHashCode."](https://learn.microsoft.com/en-us/archive/blogs/ericlippert/guidelines-and-rules-for-gethashcode) I find a couple rules essential: keeping hash codes stable, not using them as identifiers and making sure hash code calculations are fast.

### Keep Hash Codes Stable

It's critical that you keep hash codes the same for the immutable state of an object. If you include mutable state, it may have unintended consequences. For example, let's say we have a class called MutableData, as shown in Listing 4.

*Listing 4 - Defining Mutable Fields*

```c#
public sealed class MutableData
{
  public override int GetHashCode() => 
    this.Value.GetHashCode();

  public Guid Value { get; set; }
}
```

Now, imagine that we have a read-only `MutableData` property on a `Person` type, where the hash code value of `MutableData` is used in concert with others to calculate the hash code value of `Person`. We'll talk about how to do this properly in the "Approaches to Hash Code Calculations" section below, but for now we'll stick with a simple approach using the XOR operator, as shown in Listing 5.

*Listing 5 - Using XOR To Calculate Hash Codes*

```c#
public sealed class Person
{
  public Person(uint age, MutableData data, Guid id, string name) =>
    (this.Age, this.Data, this.Id, this.Name) =
        (age, data, id, name);

  public override int GetHashCode() =>
    this.Age.GetHashCode() ^
    this.Data.GetHashCode() ^
    this.Id.GetHashCode() ^
    this.Name.GetHashCode();

  public uint Age { get; }
  public MutableData Data { get; }
  public Guid Id { get; }
  public string Name { get; }
}
```

Now, in Listing 6, let's add a person to a hash set, change `Value` on the `Data` property and see if we can still find that object.

*Listing 6 - Finding Objects With Modified State*

```c#
var people = new HashSet<Person>();
var person = new Person(
  25, new MutableData { Value = Guid.NewGuid() },
  Guid.NewGuid(), "Fred");
people.Add(person);
Console.Out.WriteLine(people.Contains(person));
person.Data.Value = Guid.NewGuid();
Console.Out.WriteLine(people.Contains(person));
```

With this code, you'll see `true` in the console window, but the second call to `Contains()` will return `false`. Why? The collection assumes that the hash code for the given person will always be the same, so it uses that value when it creates its internal tables for lookup purposes. However, when we change the Value property on the `MutableData` instance, the hash code value will change, so the second call to `Contains()` won't find the object that we know is in the collection.

For that reason, you need to be careful with the data you use to create an object's hash code. It's common that hash codes will be cached, which means if you change the hash code for an object, there be dragons!

### Hash Codes Aren't Identifiers

While you can cache a hash code for hash tables, never use a hash code to identify an object. Hash codes should stay the same for the lifetime of an object in an application's execution. However, the computed hash code value may be different the next time you run the application. For that reason, don't store the hash code values for use the next time the application runs. Remember that hash codes are for putting objects in hash tables and nothing else.

### Hash Code Calculations Should Be Fast

You're creating a numeric value to represent the state of an object, and that calculation will run every time you add an object to a collection that uses hash codes. This shouldn't take a lot of time to run. Of course, as soon as a developer hears something should be fast, they'll ask, "How fast?" Fair question. Generally, hash code calculations should be in the nanoseconds range for modern hardware. If your code takes seconds to create a hash code, you should work hard to drastically cut that time down. But don't despair! In the next section, I'll cover a couple modern and performant approaches to hash code creation, which will help you avoid this lag.

## Approaches To Hash Code Calculations

When .NET first came out, I found little to no guidance on how to calculate a hash code such that these values had good distribution and performed well. The XOR technique I shared earlier in this article was one of the first I used. However, it has its share of issues, including producing zero values more often than desired.
We can also create hash codes by using two prime numbers in an iterative fashion, as demonstrated in Listing 7.

*Listing 7 - Using Prime Numbers in Hash Code Calculations*

```c#
public override int GetHashCode()
{
  var hash = 17;
  hash = hash * 23 + this.Age.GetHashCode();
  hash = hash * 23 + this.Id.GetHashCode();
  hash = hash * 23 + 
    this.Name.GetHashCode(StringComparison.CurrentCulture);
  return hash;
}
```

[There are many other techniques](https://stackoverflow.com/questions/263400/what-is-the-best-algorithm-for-overriding-gethashcode) to create hash codes. However, as a developer, this isn't something I want to concentrate on. Can't we just create something that generates good hash code values while I work on more interesting problems?

I've found a couple of ways to generate a hash code with very little work. The first, as shown in Listing 8, is to create a tuple, and call `GetHashCode()`.

*Listing 8 - Using Tuples for Hash Code Creation*

```c#
public override int GetHashCode() => 
  (this.Age, this.Id, this.Name).GetHashCode();
```

It's interesting how the hash code is calculated. For a tuple with one value, `GetHashCode()` just returns the hash code of that value. But for two or more values in the tuple, `GetHashCode()` uses `HashCode.Combine()`. Why go through a layer of indirection when you can just call `HashCode.Combine()` directly? As shown in Listing 9, you might as well just use `HashCode`.

*Listing 9 - Using `HashCode.Combine()`*

```c#
public override int GetHashCode() => 
  HashCode.Combine(this.Age, this.Id, this.Name);
```

As a side note, `HashCode` uses [this code](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/HashCode.cs) to generate a hash code, which is based on code found in [this repository](https://github.com/Cyan4973/xxHash/). If you're interested in diving into the details of how `HashCode.Combine()` works, I recommend reviewing the code at these two links.

The new C# 9.0 [records](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history#c-version-9) feature also supports hash code creation out-of-the-box in an interesting way. Let's create a record-based `Person` as shown in Listing 10:

*Listing 10 - Defining Types Using Records*

```c#
public record Person(
  uint Age, MutableData Data, Guid Id, string Name);
```

Once compiled, open it in a decompiler like [ILSpy](https://github.com/icsharpcode/ILSpy). The generated code for `GetHashCode()` should look similar to the code in Listing 11.

*Listing 11 - `GetHashCode()` For Records*

```c#
public override int GetHashCode()
{
  return (((EqualityComparer<Type>.Default.GetHashCode(EqualityContract) * -1521134295 + 
    EqualityComparer<uint>.Default.GetHashCode(Age)) * -1521134295 + 
    EqualityComparer<MutableData>.Default.GetHashCode(Data)) * -1521134295 + 
    EqualityComparer<Guid>.Default.GetHashCode(Id)) * -1521134295 + 
    EqualityComparer<string>.Default.GetHashCode(Name);
}
```

Note that the generated implementation doesn't use `HashCode.Combine()`. This is because if the C# team used the `HashCode` class, the record feature would be tied to specific versions of .NET platforms. By not taking that dependency, they've made it possible to use the record feature on down-level platforms.

Note that this code also uses the `MutableData` field in its calculation. Hash code computations should only use immutable values. Figuring out which property types are immutable and which aren't is impossible (or at the very least, really hard), so the code responsible for generating `GetHashCode()` for a record defaults to including all defined fields. Records are meant to be immutable by default, so if you throw in mutable properties or properties defined with mutable types, you'll have to override `GetHashCode()` yourself and implement it correctly.

There are other ways to generate hash codes. In fact, the `string` type uses an internal type called [`Marvin`](https://source.dot.net/#System.Private.CoreLib/src/runtime/src/libraries/System.Private.CoreLib/src/System/Marvin.cs) to generate its hash code. For most types, using something like `HashCode.Combine()` will be more than sufficient. However, it's always helpful to see other approaches that may be useful in specific cases.

## Which Implementation Is The Best?

I mentioned that hash code generation should be fast, which begs the question: Which approach is the best? We want to use an implementation that is fast and minimizes memory allocation. I used [Benchmark.NET](https://benchmarkdotnet.org/) to test the algorithms I mentioned, and you can see the results in Listing 12.

*Listing 12 - Benchmark Results For Hash Code Creation*

|                                  Method |       Mean | Allocated |
|---------------------------------------- |-----------:|----------:|
|    GetHashCodeFromHashCodeCombinePerson |  11.803 ns |         - |
|      GetHashCodeFromTupleHashCodePerson |  21.135 ns |         - |
|        GetHashCodeFromXORHashCodePerson |   9.712 ns |         - |
| GetHashCodeFromCalculatedHashCodePerson | 214.938 ns |      32 B |
|             GetHashCodeFromRecordPerson |  20.883 ns |         - |

Keep in mind that these are results for a specific case of hash code creation - using three fields that are a `string`, a `Guid`, and an `uint`. That said, the route of using `HashCode.Combine()` is the way to go. It's easy to use, it's performant and it generates good hash code values.

## Conclusion

Hash codes are prevalent in .NET. Fortunately, in modern applications, it's easy to create these codes using common implementations that meet all the expectations of good hash code generation. In some cases, as with records, the compiler will generate that code for you, so you don't need to do anything. As I mentioned in my last article, source generators allow you to build code during the compilation phase. It's feasible in C# 9.0 and beyond to create generators that will create custom hash codes based on your needs. Happy coding!

> Published: 12.22.2020