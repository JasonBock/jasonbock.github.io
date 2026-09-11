---
title: Generating Code in C#
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Generating Code in C#

## Abstract

A new feature in .NET 5 that will be available in C# 9.0 (which is slated for [release in November 2020](https://devblogs.microsoft.com/dotnet/introducing-net-5/)) is called "source generators." It provides a means to create code based on expected conditions in existing code. This feature is baked into the compiler, creating a seamless, native code generation experience. In this article, I'll dive into source generators and demonstrate how you can use them for your own implementation needs.

## Introduction

As developers, we enjoy the opportunity to control all the hardware on a machine so the lives of those who use the applications we create are made better. We automate redundant tasks and simplify complex flows. Engineering these applications sometimes leads to conditions where we must rewrite code we've created before, repeatedly. We use techniques to reuse code when it makes sense to minimize that churn, but we often run into scenarios where that approach isn't feasible. This can lead to code that doesn't run as performant as it could, or requires developers to remember conventions and patterns that must be implemented in a precise manner. Let's look at a couple of examples to illustrate this situation.

## Implementing Equality

To start, let's look at what it takes to implement equality for a type. Consider the simplistic definition of a person shown in Listing 1.

*Listing 1 - Simple Definition Of A Person*

```c#
public sealed class Person
{
  public Person(uint age, string name) =>
    (this.Age, this.Name) = (age, name);

  public uint Age { get; }
  public string Name { get; }
}
```

If we wanted to compare two `Person` objects, we'd want to compare their `Age` and `Name` property values. But there's a lot of ceremony a developer needs to implement equality:

* You must override `Equals()` and `GetHashCode()`
* You must implement `IEquatable<T>`
* You should override the `==` and `!=` operators

In Listing 2, you can see that what was a simple definition is now a bit more complicated.

*Listing 2 - Definition Of A Person With Equality*

```c#
public sealed class Person
  : IEquatable<Person?>
{
  public Person(uint age, string name) =>
    (this.Age, this.Name) = (age, name);

  public uint Age { get; }
  public string Name { get; }

  public override bool Equals(object? obj) =>
    this.Equals(obj as Person);

  public bool Equals(Person? other) =>
    other is not null &&
      this.Age == other.Age &&
      this.Name == other.Name;

  public override int GetHashCode() =>
    HashCode.Combine(this.Age, this.Name);

  public static bool operator ==(Person? left, Person? right) =>
    EqualityComparer<Person>.Default.Equals(left, right);

  public static bool operator !=(Person? left, Person? right) =>
    !(left == right);
}
```

Implementing equality isn't hard, but it's cumbersome, time-consuming and error prone. What if the developer is having a bad day and flips the Boolean logic in one place? As developers, we want to automate all things for many reasons - to minimize human error, to create efficiencies, and because the more we can move to mechanized processes, the more we can focus on interesting and pressing problems.

Note that the code in Listing 2 is repeatable. No matter what type is given, the code could be created through a tool. In fact, Visual Studio has a refactoring to do just that.

*Using the "Generate Equals and GetHashCode" Visual Studio Refactoring*

![Generating Code](https://jasonbock.net/images/Generating-Code-1.png "Generating Code")

When you select this option, a dialog box will appear, allowing you to select different options for implementing equality, such as selecting the properties that will be used to determine it and whether or not you want the `!=` and `==` operators to be implemented.

![Generating Code](https://jasonbock.net/images/Generating-Code-2.png "Generating Code")

This is how the equality code for Person in Listing 2 was created.

On a related note, records, a new feature coming in C# 9.0, essentially does this all for you. Let's create Person as a record:

```c#
public record Person(uint Age, string Name);
```

Now, let's load the assembly that contains `Person` into a decompiling tool like [ILSpy](https://github.com/icsharpcode/ILSpy). You'll see equality is automatically implemented for you.
 
![Generating Code](https://jasonbock.net/images/Generating-Code-3.png "Generating Code")

Unfortunately, the work to handle equality with records happens within the compiler itself. If only there was a way to automate equality ... maybe by adding an attribute to a class as shown in Listing 3:

*Listing 3 - Equatable Attribute*

```c#
[Equatable]
public partial sealed class Person
{
  public Person(uint age, string name) =>
    (this.Age, this.Name) = (age, name);
  public uint Age { get; }
  public string Name { get; }
}
```

The `[Equatable]` attribute acts as a marker for a tool to find and generate the equality code within another partial class. Nothing else needs to be done by the developer. It would also work independent of a specific integrated development environment (IDE). While automatic implementation of equality sounds intriguing, we can't do this right now in C#. There's no native mechanism in place that allows a developer to read the contents of code and build more based on what currently exists.

## Implementing `ToString()`

Automating the generation of repeatable code is a desirable feature, but there's another aspect to generating code, which is to implement performant applications.
Let's say you wanted to provide an informative description of `Person` via `ToString()`:

```c#
public override string ToString() =>
  $"Age = {this.Age}, Name = {this.Name}";
```

The pattern of using public, readable properties to generate name/value pairs separated by a comma is easy to describe. It's also easy to forget to do, and a developer may make a mistake implementing it. One alternative approach is to use Reflection, as shown in Listing 4.

Listing 4 - Using Reflection to Implement ToString()

```c#
public static class ObjectExtensions
{
  public static string GetString(this object self) =>
    string.Join(", ",
      self.GetType().GetProperties(
        BindingFlags.Instance | BindingFlags.Public)
        .Where(_ => _.CanRead)
        .Select(_ => $"{_.Name} = {_.GetValue(self)}"));
}
```

This will work for any type in .NET. You just need to override `ToString()` like this:

```c#
public override string ToString() =>
  this.GetString();
```

There are two issues with this. First, putting extension methods on the object type isn't something you should encourage, primarily because these methods can't be used as normal extension methods in Visual Basic (though this problem is probably one few people will run into - see Section 5.6 of the ["Framework Design Guidelines, 3rd Edition"](https://www.oreilly.com/library/view/framework-design-guidelines/9780135896457/) for more details). The other, more pressing, concern is that Reflection adds overhead. I ran performance tests via [Benchmark.NET](https://benchmarkdotnet.org/) between the two `ToString()` approaches, and the Reflection solution is four times slower. It consumes four times as much memory as well. You can minimize that overhead through other approaches, such as creating expression trees, which are compiled, executed and cached at runtime, but these are cumbersome.

The ideal approach is to automate the `ToString()` implementation so that a developer can write code like this:

```c#
[ToString]
public partial class Person { ... }
```

Just like the `[Equatable]` attribute idea, a code generator can look for the presence of `[ToString]` in code and implement `ToString()` method in another partial class in a performant manner.

We've covered two cases, object equality and overriding `ToString()`, which can be deferred to an automated process. In the next section, I'll demonstrate how source generators work.

## Generating Code

Generating code isn't a new concept. Developers use code generators in multiple ways, either through simplistic approaches such as string building or by using existing tools or ones they create. For example, [T4](https://learn.microsoft.com/en-us/visualstudio/modeling/code-generation-and-t4-text-templates) is a tool that leverages a template engine to create code. [Scriban](https://github.com/scriban/scriban) is another one. The issue is none of these have native integration with the underlying C# compiler. With the scenarios described in the previous section, changes to the Person type might affect the equality and ToString() implementations. Having the code regenerated automatically reduces any discrepancies and errors. Coupled with good IDE integration, the developer can also see the generated code and immediately understand what's going on.

### What Are Source Generators?

A source generator, [as defined by Microsoft](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/), is "a piece of code that runs during compilation and can inspect your program to produce additional files that are compiled together with the rest of your code."

Essentially, you read the contents of the current compilation, which means searching through syntax trees to find nodes, tokens and (possibly) symbols for specific conditions. If they exist, you create new C# code that's included as part of the compilation process.

Before I show you an example of a source generator, keep in mind you can't edit existing code. This means you can't change the body of a method or remove properties from a class; you can only add new code.

### Creating Object Mappers

The area we're going to tackle with source generation is object mapping. The idea is simple: Take two objects that may or may not have any kind of type relationship and map the matching property values from one to another. For example, in Listing 5, we have two types, a source type, and a destination type.

*Listing 5 - Defining Mappable Classes*

```c#
public sealed class Source
{
  public decimal Amount { get; set; }
  public Guid Id { get; set; }
  public int Value { get; set; }
  public string? Name { get; set; }
}

public sealed class Destination
{
  public Guid Id { get; set; }
  public int Value { get; set; }
  public string? Name { get; set; }
}
```

To map the two types, we would need to ignore the `Amount` property from the `Source` object, as shown in Listing 6.

*Listing 6 - Mapping The Source To The Destination*

```c#
var source = new Source
{
  Amount = 33M,
  Id = Guid.NewGuid(),
  Value = 10,
  Name = "Woody"
};

var destination = new Destination
{
  Id = source.Id,
  Value = source.Value,
  Name = source.Name
};
```

We can automate creating this mapping code. Further, we want to map the properties directly rather than use a Reflection-based approach, which wouldn't be as fast. So, let's create an object-mapper to create an extension method off the source type that returns a destination type with the properties set correctly. I've created a NuGet package called [InlineMapping](https://www.nuget.org/packages/InlineMapping/) to do just this. You can find this code in [this repository](https://github.com/jasonbock/inlinemapping).

Let's walk through the specific steps to create a source generator for object mapping.

First, you need to create a class that will participate in the generation process. You do this by attributing the class with `[Generator]`. You also need to implement the `ISourceGenerator` interface, as shown in Listing 7.

*Listing 7 - Implementing The ISourceGenerator Interface*

```c#
[Generator]
public sealed class MapToGenerator
  : ISourceGenerator
{
  public void Execute(GeneratorExecutionContext context) { ... }
  public void Initialize(GeneratorInitializationContext context) { ... }
}
```

The `Initialize()` method is primarily there to allow you to do filtering on the syntax nodes coming through the compilation pipeline. In the case outlined, we'll need to look for classes defined with a specific attribute, which I'll show in a moment. If you don't need this feature, simply make `Initialize()` do nothing.

To put this filter in place, we register an action with `RegisterForSyntaxNotifications()`:

```c#
public void Initialize(GeneratorInitializationContext context) =>
  context.RegisterForSyntaxNotifications(() => new MapToReceiver());
```

The `MapToReceiver` class implements the `ISyntaxReceiver` interface. This has one method, `OnVisitSyntaxNode()`, where you can look at a syntax node within the compilation to determine if it's of interest. Listing 8 shows how it's implemented for object mapping.

*Listing 8 - Defining A Syntax Receiver*

```c#
public sealed class MapToReceiver
  : ISyntaxReceiver
{
  public List<TypeDeclarationSyntax> Candidates { get; } =
    new List<TypeDeclarationSyntax>();

  public void OnVisitSyntaxNode(SyntaxNode syntaxNode)
  {
    if(syntaxNode is TypeDeclarationSyntax typeDeclarationSyntax)
    {
      foreach (var attributeList in 
       typeDeclarationSyntax.AttributeLists)
      {
        foreach (var attribute in attributeList.Attributes)
        {
          if(attribute.Name.ToString() == "MapTo" ||
            attribute.Name.ToString() == "MapToAttribute")
          {
            this.Candidates.Add(typeDeclarationSyntax);
          }
        }
      }
    }
  }
}
```

We're looking for types that have an attribute called `MapToAttribute`. It specifies the type we want to map to. For our `Source` class, we add this attribute to state that we want to map to the `Destination` type:

```c#
[MapTo(typeof(Destination))]
public class Source { ... }
```

Types that have this attribute are stored in the `Candidates` list, which is further examined within the generator. You want to do this to reduce the amount of analysis that needs to be done within the generator itself, where you'll probably use symbols to determine what code should be generated. Using a syntax receiver streamlines this process.

All the real work needs to occur in the `Execute()` method. Let's start by creating the `MapToAttribute` type:

```c#
var (mapToAttributeSymbol, compilation) =
  Assembly.GetExecutingAssembly().LoadSymbol(
    "InlineMapping.MapToAttribute.cs",
    "InlineMapping.MapToAttribute", context);
```

The `LoadSymbol()` method reads the code stored in the `InlineMapping.MapToAttribute.cs` resource and adds it to the current compilation. This may seem odd at first glance - why don't I just put the file that contains `MapToAttribute` in the project and compile it like any other .cs file? Because the resulting assembly, referenced via a `ProjectReference` or a `PackageReference`, needs to be referenced as an analyzer assembly. In this case, the types within that assembly are used strictly in the compilation process, and they're not available from the referencing assembly. By adding the type into the compilation, the developer will be able to "see" `MapToAttribute`.

The rest of the `Execute()` implementation reads the types from the `Candidates` list in the `MapToReceiver` instance, finds all the `MapToAttribute` values on the type and generates the mapping methods:

*Listing 9 - Implementing Execute()*

```c#
if (context.SyntaxReceiver is MapToReceiver receiver)
{
  foreach (var candidateTypeNode in receiver.Candidates)
  {
    var model = compilation.GetSemanticModel(
      candidateTypeNode.SyntaxTree);
    var candidateTypeSymbol = model.GetDeclaredSymbol(
      candidateTypeNode) as ITypeSymbol;
    if (candidateTypeSymbol is not null)
    {
      foreach (var mappingAttribute in 
       candidateTypeSymbol.GetAttributes()
        .Where(
          _ => _.AttributeClass!.Equals(
            mapToAttributeSymbol, SymbolEqualityComparer.Default)))
      {
        var (diagnostics, name, text) =
          MapToGenerator.GenerateMapping(
            candidateTypeSymbol, mappingAttribute);

        foreach (var diagnostic in diagnostics)
        {
          context.ReportDiagnostic(diagnostic);
        }

        if (name is not null && text is not null)
        {
          context.AddSource(name, text);
        }
      }
    }
  }
}
```

The implementation of `GenerateMapping()` isn't trivial. I won't go through every line in this article, but I'll cover the key areas. The first thing I check for is the existence of a no-argument, public constructor:

```c#
var diagnostics = ImmutableList.CreateBuilder<Diagnostic>();

var destinationType =
  (INamedTypeSymbol)attributeData.ConstructorArguments[0].Value!;

if (!destinationType.Constructors.Any(
  _ => _.DeclaredAccessibility == Accessibility.Public &&
    _.Parameters.Length == 0))
{
  diagnostics.Add(Diagnostic.Create(
    new DiagnosticDescriptor(...)));
}
```

What's nice about syntax generators is that you can create diagnostics to report when conditions arise in code that you want the user to know about. In this case, if we can't create an instance of the destination object, we create a new `DiagnosticDescriptor` stating that an accessible constructor is needed to create the mapping.

Next, we create a list of the properties on the source and destination objects and look for matches. For each property on the source object that has a visible getter, is there a property on the destination object that has the same name and type with a visible setter? If so, create the C# code that will map the values:

```c#
maps.Add(
  $"\t\t\t\t\t{destinationProperty.Name} = self.{sourceProperty.Name},");
```

Once we run through all the properties, if we found any that can be mapped, we create a static class in the same namespace as the source type and generate an extension method called `MapTo{DestinationTypeName}`. This contains the mapping code. In our case, it will be called `MapToDestination()`. Fortunately, the 16.8 preview 3.1 version of Visual Studio 2019 supports "Go To Definition" for generated code. Let's write code that uses the generated extension method, as seen in Listing 10:

*Listing 10 - Using Generated Mapping Code*

```c#
var source = new Source
{
  Amount = 33M,
  Id = Guid.NewGuid(),
  Value = 10,
  Name = "Woody"
};

var destination = source.MapToDestination();
```

Using "Go To Definition" on `MapToDestination()` shows this:

```c#
using System;

namespace SourceNamespace
{
  public static partial class SourceMapToExtensions
  {
    public static Destination MapToDestination(this Source self) =>
      self is null ? throw new ArgumentNullException(nameof(self)) :
        new Destination
        {
          Id = self.Id,
          Value = self.Value,
          Name = self.Name,
        };
  }
}
```

We throw an `ArgumentNullException` if the source reference is `null` (note that the generator won't add this null check if the source type is a value type). Otherwise, the mapping code is exactly as we expect.

I'll confess, I was very happy the first time I got this to work. Having the power to synthesize code like this on the fly is amazing! A developer states they want a source type to map to a destination type, and the mapping is automatically generated in a performant way. I stress "performant," because problems like this have been typically handled through Reflection-like techniques, which are not performant. I created a project in the InlineMapping solution called InlineMapping.PerformanceTests, which compares it to an implementation using Reflection, as well as another method using [AutoMapper](https://automapper.org/), a popular .NET mapping package. Here are the results:

|             Method |        Mean |  Ratio |  Gen 0 | Gen 1 | Gen 2 | Allocated |
|------------------- |------------:|-------:|-------:|------:|------:|----------:|
|     MapUsingInline |    11.95 ns |   1.00 | 0.0153 |     - |     - |      64 B |
| MapUsingReflection | 2,375.37 ns | 201.25 | 0.3128 |     - |     - |    1320 B |
| MapUsingAutoMapper |   132.42 ns |  11.54 | 0.0248 |     - |     - |     104 B |

By generating the code as part of the compilation process, we can generate efficient code that doesn't consume a lot of memory. On a related note, if you can find even more gains in what I did, I will happily consider pull requests!

## Other Examples

My InlineMapping example is one example of how you can use source generators in C#. As I mentioned, there's already a framework out there called AutoMapper that's extremely popular (at the time I'm writing this article, the package has over 100 million downloads). It's conceivable that package's implementation could be changed to use source generators, making it even faster. Plenty of other examples show where the power of source generators come into play:

* [StrongInject](https://github.com/YairHalberstadt/stronginject) - A performant, compile-time checked inversion-of-control (IoC) container
* [Rocks](https://github.com/JasonBock/Rocks) - This is my other source generator project, a package that creates mocks for tests. I'm currently changing it so the mocks are generated at compile time. You can follow the work through [this issue](https://github.com/JasonBock/Rocks/issues/121).

I strongly encourage you to peruse these examples. Maybe they will inspire you to use source generators in other areas.

## Conclusion

Source generators are a powerful feature coming to C# 9.0. With this feature, you can modernize repetitive coding patterns in a safe, performant manner. I encourage you to give it a try. You may be surprised how much you can accomplish when you generate code. Happy coding!

> Published: 10.29.2020