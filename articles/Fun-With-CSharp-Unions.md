---
title: Fun With C# Unions
layout: default
---
| [Home](Index.html) | [Biography](Biography.html) | [Speaking](Speaking.html) | [Articles](Articles.html) | [Books](Books.html) | [Music](Music.html)

# Fun With C# Unions

Recently I've been playing with unions, a [new feature](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/union) coming in C# 15. Unions have been requested for a **long** time, and they're finally coming at the end of 2026. If you haven't seen it yet, here's what a union looks like and how it behaves:

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

This says that `Pet` can hold an instance of a `Cat`, or `Dog`, or `Bird`. Unions are also closed types, which means that you don't need a default case in the `switch` expression, which is kind of nice.

Whenever a new feature is added to C# or .NET, it can affect libraries that I support. [Rocks](https://github.com/JasonBock/Rocks), a mocking framework I've written that uses a source generator to create all the mocking infrastructure, is a prime example of this. It seems like with every new version, **something** breaks what I've done. This has been a great way for me to learn about features that I wasn't aware of, or how a new feature works. This is evident with [CslaGeneratorSerialization](https://github.com/JasonBock/CslaGeneratorSerialization), a custom serializer for [CSLA](https://github.com/MarimerLLC/csla) that uses a source generator. It's been "fun" trying to figure out how to [support a union type](https://github.com/JasonBock/CslaGeneratorSerialization/issues/49), and, in the process, I've found some interesting stuff related to unions. It's been a good exercise to try "what if?" ideas with a new feature and see what that does to break simplistic assumptions.

## One Case Unions

You can create unions with one type case:

```c#
public union Simple(Cat);
```

I'm not sure **why** you'd do this. It kind of feels like a mistake. Why create a union with just one type? There may be a valid case to do this, but right now, I'm kind of struggling wondering why someone would define a union this way. Just keep in mind that the compiler allows this, so always think of a union as having "one-of-many" type cases, where "many" can be "one". However, "many" can't be "zero" - in other words, a union always needs a case type. This won't compile:

```c#
// Fails with CS1031 - "Type expected"
public union Simple();
```

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

## Custom Union Types

Unions are a `struct` underneath the scenes with constructors that have one parameter, representing each type that can be assigned to the union. If you're interested, you can create your own union your own way, so long as your implementation [conforms to standards](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/union#custom-union-types). My bet will be that the majority of developers will just use the `union` keyword and only create custom implementations for union-like types in existing libraries.

The real concern here is knowing how to understand the parts of a union without having to worry about all these details. If you need to get all the type cases that a union supports, avoid creating your own implementation to calculate the case types. There's two areas in .NET where you can do introspection of a union type, and it's good to know what's there to help you out (or leave you hanging):

* Compiler API: There's a `IsUnion` property on `ITypeSymbol`, but unfortunately there isn't a member to get you the case types ... yet. It appears that in Preview 8 (or whatever comes after Preview 7) a `UnionCaseTypes` property that returns `ImmutableArray<ITypeSymbol>` will be added (see [this link](https://github.com/dotnet/roslyn/pull/84707) for the associated PR). My assumption is that this will get the correct list, whether the union type was defined with `union` or `IUnionMembers` is in play.
* Reflection: As far as I can tell, there is nothing off of a `Type` to tell you if it's a union or what the case types are. I might be missing it, and maybe it'll be there before .NET 11, but then again, maybe it won't be added. A `Type` is for .NET and it isn't C#-specific. Keep in mind that F# has its' own union type called a [*discriminated union*](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions), and that isn't reflected (no pun intended) in a `Type` either. I'm sure someone will create an extension method for `Type` that will do the right things (i.e. look for the `[Union]` attribute, determine if `IUnionMembers` exists, etc.).

## What's The Case Type Limit?

The more I messed with unions, I started to listen to the intrusive thoughts and I wondered ... just how many types can a union contain? So, I wrote a small program that lets me create an experiment:

```c#
const int TypeCount = 10_000;
var unionCaseTypes = 
  string.Join(',', Enumerable.Range(0, TypeCount).Select(i => $"T{i}"));
var caseTypeDefinitions = 
  string.Join(' ', Enumerable.Range(0, TypeCount).Select(i => $$"""public class T{{i}}{}"""));
var source = $"public union T({unionCaseTypes}); {caseTypeDefinitions}";
var syntaxTree = CSharpSyntaxTree.ParseText(source, new CSharpParseOptions(LanguageVersion.Preview));

var compilation = CSharpCompilation.Create("generator", [syntaxTree],
  AppDomain.CurrentDomain.GetAssemblies()
    .Where(_ => !_.IsDynamic && !string.IsNullOrWhiteSpace(_.Location))
    .Select(_ => MetadataReference.CreateFromFile(_.Location)),
  new CSharpCompilationOptions(
    OutputKind.DynamicallyLinkedLibrary));

using var outputStream = new MemoryStream();
var result = compilation.Emit(outputStream);

Console.WriteLine($"Was emit successful? {result.Success}");
```

What this C# code is doing is defining a union based on a number of case types. If `TypeCount` was 4, `source` would look like this:

```c#
public union T(T0,T1,T2,T3); public class T0{} public class T1{} public class T2{} public class T3{}
```

Changing the value of `TypeCount` let me try more and more case types. It took about 10 to 20 seconds on my machine to get this compilation to emit successfully with 10,000 case types. I tried 100,000, and I stopped it after getting bored waiting.

Now, realistically, developers will define unions based off a small handfull of types, like 3 or 4. In my opinion, if you add more, it feels like having too many method parameters. Like, why would anyone define a method with 70 parameters? Yes, I've seen this, and worse, in my career.

If there is a theoretical limit, I don't know what it is, but I really hope no one reaches it.

## Summary

At first glance, unions seem straightforward, and for the vast majority of cases where a union scratches an itch, it'll be just fine. But it's always fun to dive in and see what works and what doesn't, and hopefully my spelunking shed some light on some corner-ish cases you may have been thinking of.

> Published: 08.12.2026 8:00 AM CST