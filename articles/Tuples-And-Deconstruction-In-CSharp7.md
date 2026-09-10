---
title: Tuples and Deconstruction in C#7
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Tuples and Deconstruction in C#7

Synopsis
With the addition of tuples in C#7 comes the ability to deconstruct them into their constituent parts. In this article, you'll see how far you can push this feature to allow any type to be deconstructed.

## The Basics of Deconstruction

What is deconstruction? It's the ability to split a tuple's values up and put them into variables. For example, here's how you take a tuple with a `string` and `int` and deconstruct them into variables:

```c#
var tuple = (Name: "Jim", Age: 32);
var (tupleName, tupleAge) = tuple;
Console.Out.WriteLine(
  $"{nameof(tuple)} - {tupleName} is {tupleAge}");
```

This is beneficial if a method returns a tuple that didn't name the tuple's values. Instead of doing this (which leaves your code base with names like `Item1` and `Item2`):

```c#
private static (string, int) GetPerson()
{
  return ("Joy", 15);
}

var person = GetPerson();
Console.Out.WriteLine(
  $"{nameof(GetPerson)} - {person.Item1} is {person.Item2}");
```

You can deconstruct the tuple into variables with good names:

```c#
var (name, age) = GetPerson();
Console.Out.WriteLine(
  $"{nameof(GetPerson)} - {name} is {age}");
```

## Supporting Deconstruction

For the underlying tuple type in C#7, `System.ValueTuple`, deconstruction comes for free because the compiler knows how to take that tuple and deconstruct its data into the target variables. If you want your classes to support deconstruction, you have to provide an implementation of a deconstruction method yourself. This method must be called `Deconstruct()` and it must exist either as an instance method on the source type or an extension method that takes the source type as the first parameter (i.e. the `this` parameter). For example, here's an example of a type with `Deconstruct()` defined on the type:

```c#
public sealed class PersonWithDeconstruct
{
  public PersonWithDeconstruct(string name, int age)
  {
    this.Name = name;
    this.Age = age;
  }

  public void Deconstruct(out string name, out int age)
  {
    name = this.Name;
    age = this.Age;
  }

  public string Name { get; }
  public int Age { get; }
}

var person = new PersonWithDeconstruct("Jeff", 12);
var (name, age) = person;
Console.Out.WriteLine($"{nameof(person)} - {name} is {age}");
```

The compiler will end up calling the `Deconstruct()` method on the line of code after person has been created.

If you don't own the type definition, you can add an extension method to support deconstruction:

```c#
public sealed class PersonWithoutDeconstruct
{
  public PersonWithoutDeconstruct(string name, int age)
  {
    this.Name = name;
    this.Age = age;
  }

  public string Name { get; }
  public int Age { get; }
}

public static class PersonWithoutDeconstructExtensions
{
  public static void Deconstruct(
    this PersonWithoutDeconstruct @this, out string name, out int age)
  {
    name = @this.Name;
    age = @this.Age;
  }
}

var person = new PersonWithoutDeconstruct("Jeff", 12);
var (name, age) = person;
Console.Out.WriteLine($"{nameof(person)} - {name} is {age}");
```

This is how the original `System.Tuple` type, added to .NET in 4.0, supports deconstruction. The `TupleExtensions` static class has a number of `Deconstruct()` methods so you can deconstruct these "old" tuples:

```c#
var oldTuple = Tuple.Create("Johnny", 55);
var (oldTupleName, oldTupleAge) = oldTuple;
Console.Out.WriteLine(
  $"{nameof(oldTuple)} - {oldTupleName} is {oldTupleAge}");
```

## Flexible Deconstruction

As I started playing with tuples in C#7, I started to wonder if you could deconstruct an anonymous type, which feels a lot like a tuple. Anonymous types have been around for a while in C# - they're really easy to create:

```c#
var anonymous = new { Name = "Jane", Age = 22 };
Console.Out.WriteLine(
  $"{nameof(anonymous)} - {anonymous.Name} is {anonymous.Age}");
```

Unfortunately, anonymous types have limited use. They can only be used in the method that they are declared in - they can't be returned from the method. Given the flexibility of tuples in C#7, anonymous types may end up being used less and less in .NET applications. That said, anonymous types are widely used in current C# code bases, and they have a similar structure to tuples – they're just a class with properties that map to the names and types of values used when the anonymous type was defined. In fact, here's what that anonymous type looks like if you decompile the code using a tool like ILSpy:

```c#
public sealed class '<>f__AnonymousType0`2'<'<Name>j__TPar', '<Age>j__TPar'>
{
  private readonly '<Name>j__TPar' '<Name>i__Field';
  private readonly '<Age>j__TPar' '<Age>i__Field';
  
  public '<>f__AnonymousType0`2'(
    '<Name>j__TPar' Name,
	'<Age>j__TPar' Age)
  {
    this.'<Name>i__Field' = Name;
	this.'<Age>i__Field' = Age;
  }
  
  public '<Name>j__TPar' Name => this.'<Name>i__Field';
  public '<Age>j__TPar' Age => this.'<Age>i__Field';
}
```

This is actually pseudo-C#. The names of the class and its members are completely mangled and wouldn't be valid in C#. Also, the anonymous type also has other methods like `ToString()` and `GetHashCode()` implemented, but the point is that you end up with a type that is truly "anonymous" and contains the values you used when the type was defined.

Note that the anonymous type is not generated with a `Deconstruct()` implementation. Also, there's no way to define a `Deconstruct()` extension method for that type because you don't know what the name is. Therefore, it looks like there's no way to deconstruct anonymous types.

Or can you?

Well, I tried. Here's my definition of a generic deconstruction extension method for two values:

```c#
public static void Deconstruct<T1, T2>(this object @this, out T1 value1, out T2 value2)
{
  var properties = @this.GetType()
    .GetProperties(BindingFlags.Public | BindingFlags.Instance)
    .Where(_ => _.CanRead).ToArray();
  value1 = (T1)properties[0].GetValue(@this);
  value2 = (T2)properties[1].GetValue(@this);
}
```

Note that this isn't limited to just anonymous types; it can be used for any type you find. Also, this code is clearly not defensive in its implementation. That is, I don't check to see if the number of public, instance, readable properties on a type equals the number of out parameters to the method, and I'm also assuming that the order of the properties found by `GetProperties()` matches the order defined with the generic types in the parameters. This could easily break if `@this` has 3 properties and I defined another `Deconstruct()` method with 4 `out` parameters, or one of the properties was a type that could not be cast into the desired type specified by the developer with the generic type value. Just ignore these issues for now. We're more interested to see if the compiler will even let us use this ... but unfortunately, it won't. If we try to use it:

```c#
var anonymous = new { Name = "Jane", Age = 22 };
var (anonymousName, anonymousAge) = anonymous;
```

Sad to say, but the compiler rejects it with `CS0411`: "The type arguments for method 'ObjectExtensions.Deconstruct<T1, T2>(object, out T1, out T2)' cannot be inferred from the usage. Try specifying the type arguments explicitly." I tried declaring the variables used for deconstruction with explicit types but that doesn't help at all.

This definition will work:

```c#
public static void Deconstruct(this object @this, out object value1, out object value2)
{
  var properties = @this.GetType()
    .GetProperties(BindingFlags.Public | BindingFlags.Instance)
    .Where(_ => _.CanRead).ToArray();
  value1 = properties[0].GetValue(@this);
  value2 = properties[1].GetValue(@this);
}
```

But this is downright ugly to me. Now everything is an object and I'll end up having casts and boxing with my deconstructed variables.

Is there any hope? Well, there's one way to do it, but it requires an intermediately tuple object to be made, and you can't use the magic the C# compiler adds to automatically call a `Deconstruct()` method for you. Actually, it will use that magic, but you have to call an extension method first. Here's the solution:

```c#
public static (T1, T2) To<T1, T2>(this object @this)
{
  var properties = @this.GetType()
    .GetProperties(BindingFlags.Public | BindingFlags.Instance)
    .Where(_ => _.CanRead).ToArray();
  var value1 = (T1)properties[0].GetValue(@this);
  var value2 = (T2)properties[1].GetValue(@this);

  return (value1, value2);
}
```

The `To()` extension method lets you stuff the property values you found on the given object into a tuple, and it returns that tuple. Then C# will see that you're trying to deconstruct that tuple into individual variables, and the deconstruction magic kicks in:

```c#
var anonymous = new { Name = "Jane", Age = 22 };
var (anonymousName, anonymousAge) = anonymous.To<string, int>();
Console.Out.WriteLine(
  $"{nameof(anonymous)} - {anonymousName} is {anonymousAge}");
```

Now, while this will work, it's not desirable for two reasons. First, the implementation of `To()` has to use Reflection, and as I mentioned before, even if you make `To()` defensive, it's still fragile. If you give `To()` an object with less properties than `out` parameters, or if the types don't match, it'll fail. Even if those conditions hold, there's no guarantee you're going to map the right property values into the right variables. For example, the targeted type may have three properties typed as a string. How will you know which one will get mapped into which `out` parameter? The best you can do is rely on a heuristic. Second, if you really want to deconstruct a type, either put a `Deconstruct()` method on the type itself, or write the extension method yourself if you don't own the code that defines the type. They're easy to create and write tests for. 

## Conclusion

In this article, you saw how deconstruction is done in C#7. You learned how to create your own deconstruction methods and how far you could go with that to make a generic deconstruction method. While that generic approach isn't desirable, it's still educational (and fun!) to play with language features to understand how they work. That knowledge makes you a better developer. Until next time, happy coding!

> Published: 01.25.2017