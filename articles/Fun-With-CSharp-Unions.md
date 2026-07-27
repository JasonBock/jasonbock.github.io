---
title: Fun With C# Unions
layout: default
---
# Fun With C# Unions

Recently I've been playing with unions, a [new feature](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/union) coming in C# 15. Unions have been requested for a **long** time, and they're finally coming (though I'm sure there will be some folks who don't like the final design decisions). No matter what you think of how it works, it's here, and it looks something like this:

```c#
public record class Cat(string Name);
public record class Dog(string Identifier);
public record class Bird(string Description);

public union Pet(Cat, Dog, Bird);

var pet = new Pet(new Cat("Tank"));

var name = switch pet
{
    Cat cat => cat.Name,
    Dog dog => dog.Identifier,
    Bird bird => bird.Description,
};

Console.WriteLine(name);
```

This basically says that `Pet` can hold an instance of a `Cat`, or `Dog`, or `Bird`. Unions are also closed types, which means that you don't need a "default" case in the `switch` expression, which is kind of nice.

Whenever a new feature is added to C# or .NET, it can affect libraries that I support. [Rocks](https://github.com/JasonBock/Rocks), a mocking framework that uses a source generator to create all the mocking infrastructure, is a prime example of this. It seems like with every new version, **something** breaks what I've done. This has been a great way to learn about features that I wasn't aware of, or how a new feature works. This is evident with [CslaGeneratorSerialization](https://github.com/JasonBock/CslaGeneratorSerialization), a custom serializer for [CSLA](https://github.com/MarimerLLC/csla) that uses a source generator. It's been ... "fun" trying to figure out how to [support a union type](https://github.com/JasonBock/CslaGeneratorSerialization/issues/49), and, in the process, I've found some fun stuff that you can do with unions. I don't know if anyone will actually do this, but, again, it's a good exercise to try "what if?" ideas with a new feature and see what that does to break simplistic assumptions.

## Simple Unions

You can create unions with one type case:

```c#
public union Simple(Cat);
```

I'm not sure **why** you'd do this. It kind of feels like a mistake. Why create a union with just one type? There may be a valid case to do this, but right now, I'm kind of struggling wondering why this would someone would define a union this way.

## Unions Within Unions

This may seem obvious, but yes, you can have a union within a union:

```c#
public record class Parakeet(string Name);
public record class Hummingbird(string Name);
public record class Robin(string Name);

public union BirdUnion(Parakeet, Hummingbird, Robin);

public union Pet(Cat, Dog, BirdUnion);
```

## Recursive Unions

Just like type definitions can be recursive:

```c#
public record class R1(R2 Value);
public record class R2(R1 Value);
```

Unions can be recursive as well:

```c#
public record class A1;
public record class A2;
public record class B1;
public record class B2;

public union B(B1, B2, A);
public union A(A1, A2, B);
```

## What's The Type Limit?

The thing to keep in mind with unions is that they're a `struct` underneath the scenes with constructors that have one parameter, representing each type that can be assigned to the union. If you're interested, you can create your own union your own way, so long as your implementation [conforms to standards](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/union#custom-union-types). That said, I started to listen to the intrusive thoughts and wondered...just how many types can a union contain?

So, I wrote a small program that lets me create an experiment

{TODO: Finish}

Now, realistically, developers will define unions based off a small handfull of types, like 3 or 4. In my opinion, if you add more, it feels like having too many method parameters. Like, why would anyone define a method with 70 parameters? (Yes, I've seen this, and worse, in my career).

## Summary

{TODO: Finish}