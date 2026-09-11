---
title: Handling Unexpected Results In C#
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Handling Unexpected Results In C#

## Abstract

Having historical understanding of how a language works doesn't always mean you understand how things work in the latest version. As a 20-year-old language, C# has changed a lot throughout its history. In this article, I'll delve into a specific issue I found in C# 8.0, in which common features didn't work as I expected. I'll share how this issue affected my path toward my desired solution and how I worked around it.

C# has evolved significantly over its 20-year history. The abilities to use functional programming styles, access memory in a safe manner and create default interface members allow developers to create applications for a wide variety of scenarios. Furthermore, ever since Microsoft released the [Compiler API](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/) in 2015, the [number of feature additions](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history) have increased substantially, and C# 9.0 will continue this trend when it's released near the end of 2020.

In some ways, I'm very happy to see this language continue to foster a strong community and a dedication for improvement. The way I code in C# today is vastly different from what I did years ago. That said, this comes at a price. It's harder to learn the breadth and depth of C# these days. With more features comes more effort to understand what features exist and when to use them. In addition, some well-documented existing features can suddenly (and subtly) change in surprising ways and require rethinking how you use them.

In this article, we'll look at how a new feature in C# 8.0 - nullable reference types, in combination with attributes and Reflection - works in an unexpected way. This issue cropped up in a test for a mocking framework I've been working on since 2015 called [Rocks](https://github.com/JasonBock/Rocks/). Rocks is like other mocking frameworks like [Moq](https://github.com/Moq) and [NSubstitute](https://nsubstitute.github.io/), but I don't use [Intermediate Language (IL)](https://learn.microsoft.com/en-us/dotnet/standard/managed-code#intermediate-language--execution) generation techniques to create the mock types at runtime. Rather, I generate C# code and compile it using the Compiler API, making it very easy to debug any issues I run into when I inadvertently introduce bugs in the code-generation process.

## Attributes And Reflection

To start, let's review what [attributes](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/) are in C#, how [Reflection](https://learn.microsoft.com/en-us/dotnet/api/system.reflection) works and how these two tie together. This isn't meant to be a deep dive, but I'll give you enough to have a good high-level understanding to follow the issue at hand.

### What Are Attributes?

Attributes are pieces of data associated with members in a .NET assembly. For example, developers use attributes with testing frameworks such as [NUnit](https://nunit.org/) or [xUnit](https://xunit.net/). Listing 1 shows an example of a test using NUnit:

*Listing 1. Simple NUnit Test*

```c#
using NUnit.Framework;

[assembly: Parallelizable(ParallelScope.Children)]
public static class AdderTests
{
  [Test]
  public static void Add()
  {
    // Arrange
    var x = 3;
    var y = 6;
    // Act
    var result = Adder.Add(x, y);
    // Assert
    Assert.That(result, Is.EqualTo(9));
  }
}
```

`[Test]` denotes that `Add()` should be run by NUnit when you want to run tests. Furthermore, `[Parallelizable]` states that all tests within this assembly should be run in parallel. When this code is compiled, these attributes are stored in the assembly. If you're curious, you can use tools like [ILDasm](https://learn.microsoft.com/en-us/dotnet/framework/tools/ildasm-exe-il-disassembler) or [ILSpy](https://github.com/icsharpcode/ILSpy) to inspect an assembly, and then you'll see the data that represents the assembly, as indicated in Listing 2 below:

*Listing 2. Data Representing The Assembly*

```
.method public hidebysig static
  void Add () cil managed
{
  .custom instance void
    [nunit.framework]NUnit.Framework.TestAttribute::.ctor() =
  (
    01 00 00 00
  )
  // ...
}
```

Note that `TestAttribute` is stored along with the `Add()` method. Because attributes are stored as data, they don't do anything unless code looks for them and acts accordingly. This is where Reflection comes into play.

### Using Reflection

Reflection is a set of APIs within .NET that you can use for assembly introspection. For example, Listing 3 shows how you can find all the public, static methods within the AdderTests class and print their parameter names and types:

*Listing 3. Using Reflection For Assembly Member Discovery*

```c#
var type = typeof(AdderTests);

foreach(var method in
  type.GetMethods(BindingFlags.Public | BindingFlags.Static))
{
  Console.Out.WriteLine(method.Name);

  foreach(var parameter in method.GetParameters())
  {
    Console.Out.WriteLine(
      $"{parameter.Name} - {parameter.ParameterType.Name}");
  }
}
```

Reflection isn't limited to assembly member discovery. You can invoke members like calling methods or setting properties at runtime as well. You can also generate code via types within the [System.Reflection.Emit](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.emit) namespace, though keep in mind this requires knowledge of IL.

Next, if you needed to find all NUnit tests methods in a type, Listing 4 shows one way you could do it:

*Listing 4. Using Reflection With Attributes*

```c#
var type = typeof(AdderTests);

foreach(var testMethod in type.GetMethods(
  BindingFlags.Public | BindingFlags.Static)
  .Where(_ => _.GetCustomAttribute<TestAttribute>() is { }))
{
  Console.Out.WriteLine($"{testMethod.Name} is a test method.");
}
```

To be clear, the cases that a framework like NUnit needs to handle to find and invoke all tests are far more complicated than this example, but you can use `GetCustomAttribute()` to look for methods attributed with `[Test]`. You can use other methods to find attribute data, like `GetCustomAttributes()` and `GetCustomAttributeData()`, but as you've seen, with Reflection, you can discover this metadata rather easily.

So far, so good. But, before I get into the issue I encountered, I need to cover a new C# feature: [nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types).

### Nullable Reference Types

The concept of a null value has been around since C# 1.0. It's essentially the absence of a value, but a problem arises when you accidentally use something that's set to null. For example, look at Figure 1:

*Figure 1. What Happens When You Use A Null Reference*

![Handling Unexpected](https://jasonbock.net/images/Handling-Unexpected-1.png "Handling Unexpected")

Notice the red squiggle under the name variable in the second instance of the `Length` property. When name is set to `null`, the second run will fail and throw a `NullReferenceException`. The red squiggle appears because the code exists in a project where nullable reference types have been enabled (i.e. setting `<Nullable>` in the .csproj file to enable).

Null references have been called the ["billion dollar mistake"](https://www.youtube.com/watch?v=ybrQvs4x0Ps) by [Sir Tony Hoare](https://www.cs.ox.ac.uk/people/tony.hoare/index1.html), and those who have run into issues with null values would probably agree. Finding cases where a null is dereferenced in code can be extremely difficult, and developers do their best to avoid the issue altogether. Arguably, a better approach in C# would've been to introduce constructs like an `Option<>` type, but alas, there is too much history with core .NET library design to retrofit this in, so we must deal with nulls. Moreover, we really need to be explicit with our intent in code, stating when we can tolerate null values and when we can't.

In C# 1, value types, such as int, always had a value. You couldn't assign null to a variable or parameter defined as a value type. However, there are cases where having no value is desirable. Therefore, C# 2 introduced nullable value types. To do this, you add the `?` after the type name:

```c#
public static void UseIntValue(int? value)
```

However, reference types (like `string`) have always allowed `null` as a valid value. If you find cases where your code simply cannot work with null values, the nullable reference type feature now allows you to specify when it's acceptable to use nulls:

```c#
public static void UseStringValue(string? value)
```

In the line above, I'm stating that it's acceptable to pass in a null value to `UseStringValue()`. If I instead declare the parameter type as shown in the line below, a valid value must be given for the value argument:

```c#
public static void UseStringValue(string value)
```

I could still pass in a null value, but my intention is clearly stated, and the compiler will do its best to find cases where a null value would be passed in (as you saw in Figure 1).

## Unexpected Behaviors

OK, now I can show you how attributes, Reflection and nullable reference types came together in an unexpected way. I'll demonstrate the problem I ran into and explain why I ran into this issue, as it may seem a bit obscure at first. Take a look at Listing 5:

*Listing 5. The Code That Failed*

```c#
using System.ComponentModel;
using System.Diagnostics.CodeAnalysis;

public sealed class Values
{
  public Values() =>
    (this.NullValue, this.MyValue) =
      (string.Empty, string.Empty);

  [Description("My Value")] public string MyValue { get; set; }
  [AllowNull] public string NullValue { get; set; }
}
```

Now take a look at Listing 6:

*Listing 6. Determining What Attributes Exist For A Property On A Type*

```c#
var valuesType = typeof(Values);
var myValueProperty = valuesType.GetProperty(nameof(Values.MyValue))!;
var myValuePropertyData = myValueProperty.GetCustomAttributesData();
PrintAttributeData(nameof(Values.MyValue), myValuePropertyData);
var nullValueProperty = valuesType.GetProperty(nameof(Values.NullValue))!;
var nullValuePropertyData = nullValueProperty.GetCustomAttributesData();
PrintAttributeData(nameof(Values.NullValue), nullValuePropertyData);

private static void PrintAttributeData(
  string memberName, IList<CustomAttributeData> attributes)
{
  Console.Out.WriteLine($"{memberName} - count is {attributes.Count}");

  foreach(var attribute in attributes)
  {
     Console.Out.WriteLine($"\t{attribute.AttributeType.Name}");
  }
}
```

Here, I'm using Reflection to find the `MyValue` and `NullValue` properties via `GetProperty()`, and then I call `GetCustomAttributesData()` to retrieve any associated attribute information on the property. I use the list of attribute data to print the number of attributes in the list as well as the name of each attribute.

I expected to see one attribute for each property, but instead some properties were empty, as shown in Figure 2:

*Figure 2. The Unexpected: A Property Without An Attribute*
 
![Handling Unexpected](https://jasonbock.net/images/Handling-Unexpected-2.png "Handling Unexpected")

As you can see, the `NullValue` property has no attributes, which surprised me. I can clearly see an attribute on each property, but in the second case, `[AllowNull]` is nowhere to be found. What gives?

### Where Did The Attribute Go?

Since nullable reference types were introduced, it's been very challenging to generate correct C# code that honors the intentions of the base type I'm building a mock for in Rocks. You'd think that there'd be some way to determine if a type definition has a "?" there, but it was complicated to do this. You must look for attributes that are synthesized at compile time, read the flag values, examine the context of where the type usage is and so on. (Read the ["Nullable Metadata"](https://github.com/dotnet/roslyn/blob/main/docs/features/nullable-metadata.md) document if you want the gory details.)

It didn't stop there. The .NET team also introduced nullable attributes to allow developers to specify certain nullable conditions, which may arise in their code. You can read the details [here](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/attributes/nullable-analysis), but suffice to say, `[AllowNull]` is one of those attributes. You use `[AllowNull]` when a field or a parameter has preconditions that may not match the nullable annotations on the specified type. So, why can you use `[AllowNull]` with properties?

The reason is subtle. Most .NET developers look at property definitions as "getters" and "setters," and they don't realize that properties are just syntactic sugar over methods. What really happens when you create a property in C# is something like this:

```c#
public string get_MyValue()
public string set_MyValue(string value)
```

The assembly stores information, so you don't "see" these methods when you use tools like [IntelliSense](https://learn.microsoft.com/en-us/visualstudio/ide/using-intellisense), but open an assembly in ILSpy and you'll see that these methods exist. When you use a getter property, the related `get_MyValue()` method is invoked. Correspondingly, a set property invocation takes the given value and maps it to the value parameter, which is what you see in the body of a setter property.

Given what you know now about `[AllowNull]` and properties, you can guess that the attribute is only applicable for the parameter in the setter method. In fact, if you read the documentation for this attribute, you'll see this hint:

> "You may need to add a using directive for System.Diagnostics.CodeAnalysis to use this and other attributes discussed in this article. The attribute is applied to the property, not the set accessor. The AllowNull attribute specifies pre-conditions, and only applies to inputs. The get accessor has a return value, but no input arguments. Therefore, the AllowNull attribute only applies to the set accessor."

If your hunch is that `[AllowNull]` ends up on the setter's parameter, you're right! Figure 3 shows the `NullValue` property in ILSpy:

*Figure 3. The NullValue Property in ILSpy*

![Handling Unexpected](https://jasonbock.net/images/Handling-Unexpected-3.png "Handling Unexpected")

Notice that `[AllowNull]` attribute has moved! This shift really bothers me, as I've never seen such a thing. If you look at the definition of `AllowNullAttribute`, you'll notice that its `AttributeUsageAttribute` allows it to be defined on a field, parameter or a property. With the property case, it's only valid for the setter. I'm guessing the C# team's rationale here is that developers don't view properties as methods, so it makes more sense to allow the attribute to be put on the property, and they handle the relocation to the setter's parameter behind the scenes. That said, I think it would have been correct to allow only `[AllowNull]` to be defined on either a field or a parameter. In the case of a property, it would have only been valid if a setter existed, and then the developer would have had to use `[param: AllowNull]`.

## Working Around The Issue

In Rocks, this attribute relocation issue came up in a test that finds all the types in the assembly that `System.Object` is defined in and creates mock types for any mockable types. It's a nice stress-test to ensure I'm handling member declarations that aren't typically found in assembly definitions. Recently, it failed because my mock for `TextWriter` wasn't overriding the `NewLine` property correctly. I was confused at first, because when I looked at `NewLine` via the "Go To Definition" feature in Visual Studio, I saw this:

```c#
//
// Summary:
// Gets or sets the line terminator string used by the current TextWriter.
//
// Returns:
// The line terminator string for the current TextWriter.
public virtual string NewLine { get; set; }
```

Notice that `[AllowNull]` doesn't show up, but it's there. You can see it in source, and sure enough, it's defined on the property. I needed to look at a property's setter, see if `[AllowNull]` exists and put `[AllowNull]` on the property. Finding this attribute in Reflection is straightforward, as shown in Listing 7:

*Listing 7. Using Reflection To Find `[AllowNull]`*

```c#
var nullValuePropertySetterParameterData =
  nullValueProperty.GetSetMethod()!
  .GetParameters()[0].GetCustomAttributesData();
var nullValueData = nullValuePropertyData.ToList();
nullValueData.AddRange(nullValuePropertySetterParameterData);
Program.PrintAttributeData(
  nameof(Values.NullValue), nullValuePropertySetterParameterData);
```

Once I added this check into Rocks, the issue went away.

## Conclusion

In this article, you saw how attributes can add metadata to code. Using Reflection, you can write code that performs actions based on the existence of that metadata. I covered nullable reference types, a new feature in C# 8.0, and nullable attributes, where `[AllowNull]` is handled in a special manner by the C# compiler. Therefore, you must adjust your expectations in using Reflection accordingly.

This particular issue may seem minor; however, it cost me a chunk of time trying to figure out what was going on. As far as I can tell, there's no explicit documentation that tells you that `[AllowNull]` on a property will be associated with the setter's parameter. For me, this was a reminder that no matter how well I think I know a programming language or an API, there may be surprises lurking in dark corners. Remember to always look at the source code and use disassemblers like ILSpy. These sources of truth can help guide you in addressing similar problems. Happy coding!

> Published: 08.06.2020