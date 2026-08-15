---
title: Code Injection With CCI - Part 4
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 4

Previous Parts

* [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html)
* [Part 2](https://jasonbock/articles/Code-Injection-With-CCI-Part-2.html)
* [Part 2.5](https://jasonbock/articles/Code-Injection-With-CCI-Part-25.html)
* [Part 3](https://jasonbock/articles/Code-Injection-With-CCI-Part-3.html)

In Part 3, I covered the fixup routines, which allowed us to change the IL at will and not have to worry (too much) about branches getting out of whack. In this part, I'll cover one of the injection attributes, `[NotNull]`.

Recall that `[NotNull]` is placed on a parameter to a method like this:

```c#
public AttributedCustomer(Guid id, [NotNull]int age, [NotNull]string firstName, 
  [NotNull]string lastName, BirthDate birthDate)
```

Here's how the attribute is defined:

```c#
[AttributeUsage(AttributeTargets.Parameter, AllowMultiple = false, Inherited = true)]
[Serializable]
public sealed class NotNullAttribute : Attribute
{
}
```

Let's take a look at `InjectorVisitor` to see how it handles this metadata. First, let's go over the constructors:

```c#
using CciInjector.Attributes;
using CciInjector.Extensions;
using Microsoft.Cci;
using Microsoft.Cci.MutableCodeModel;
using System.Collections.Generic;
using System.Linq;
using SR = System.Reflection;

namespace CciInjector
{
  internal sealed class InjectorVisitor : MetadataMutator
  {
    private IMethodDefinition argumentNullExceptionCtor;
    private Dictionary<uint, IOperation> exceptionMappings;
    private ITypeDefinition notNullAttribute;
    private MscorlibTypeLoader mscorlib;
    private ITypeDefinition toStringAttribute;
    private ITypeDefinition traceAttribute;

    internal InjectorVisitor(IMetadataHost host)
      : base(host)
    {
      this.LoadFields();
    }

    //  ... more to come ...
```

The two constructors simply pass on their information to `MetadataMutator`, and calls `LoadFields()`, which set the values of the fields in the object:

```c#
    private void LoadFields()
    {
      this.host.LoadAssembly(new AssemblyIdentity(
        this.host.CoreAssemblySymbolicIdentity, SR.Assembly.GetAssembly(
        typeof(object)).Location));

      this.mscorlib = new MscorlibTypeLoader(this.host);

      var injectorAttributesAssembly = this.host.LoadUnitFrom(
        "CciInjector.Attributes.dll") as IAssembly;
      var customerAssembly = this.host.LoadUnitFrom("Customers.dll") as IAssembly;

      var injectorAttributeTypes = from type in injectorAttributesAssembly.GetAllTypes()
        where AttributeHelper.IsAttributeType(type)
        select type;

      this.notNullAttribute = InjectorVisitor.Find<NotNullAttribute>(injectorAttributeTypes);
      this.toStringAttribute = InjectorVisitor.Find<ToStringAttribute>(injectorAttributeTypes);
      this.traceAttribute = InjectorVisitor.Find<TraceAttribute>(injectorAttributeTypes);

      this.argumentNullExceptionCtor = TypeHelper.GetMethod(
        this.mscorlib.GetType("System", "ArgumentNullException").ResolvedType,
        this.host.NameTable.GetNameFor(".ctor"),
        this.host.PlatformType.SystemString);
    }
```

The first thing `LoadFields()` does is loads the metadata for mscorlib.dll. This may seem really odd, especially if you've done a lot of Reflection coding in .NET before. Remember that CCI is not Reflection; it won't automatically load anything for you on startup, so you have to do that yourself. I do use Reflection to find out where mscorlib.dll resides on the machine via the `Location` property. I also load the assembly that contains the injector attributes and the `Customer` assembly that contains the base class `Customer` (I'll discuss in Part 5 why I load this explicitly).

Now I need to get the three attribute as `ITypeDefintion` references via a generic `Find()` method:
```c#
    private static ITypeDefinition Find<T>(IEnumerable<INamedTypeDefinition> types)
    {
      return (from type in types
        where type.ToString() == typeof(T).ToString()
        select type).First();
    }
```

LINQ comes to rescue here very nicely and makes the code rather succient. First, I get all of the types in my attributes assembly that are attributes in `LoadFields()` - using `AttributeHelper.IsAttributeType()` made this very easy to pull off. Originally I didn't find this helper class so I ended up coding it myself, but I'm glad I stumbled across this in the CCI API. The less I have to do, the better! Once I find all those attributes, `Find<T>()` lets me select the specific type I'm looking for. Using `ToString()` isn't the best approach, but the Name property only contains the name of the type without the namespace. Frankly, what I need to do here is the fully-qualified names for the types I discover in the standard .NET format (i.e. "full.type.name, assemblyname") but for this example `ToString()` will do (for now :) ).

Now I'm finally at the point where I can do some injection! To get the right hook in place I override one of the `Visit()` methods:
```c#
    public override MethodDefinition Visit(MethodDefinition methodDefinition)
    {
      var method = base.Visit(methodDefinition);

      this.TrackBranches(method);
      this.AddNotNullChecks(method);
      this.FixBranches(method);

      return method;
    }
```

Later on I'll add other methods to handle the other attributes, but for now let's take a look at `AddNotNullChecks()`:

```c#
    private void AddNotNullChecks(MethodDefinition method)
    {
      var notNullParameters = from paramater in method.Parameters
        where !paramater.Type.IsValueType
        where AttributeHelper.Contains(
          paramater.Attributes, this.notNullAttribute)
        select paramater;

      foreach(var notNullParameter in notNullParameters)
      {
        var operations = (method.Body as MethodBody).Operations;
        var label = new Label(operations[0]);
        var indexer = 0;

        operations.Insert(indexer++, new Operation() { 
          OperationCode = OperationCode.Ldarg, Value = notNullParameter });
        operations.Insert(indexer++, new Operation() { 
          OperationCode = OperationCode.Brtrue_S, Value = label });
        operations.Insert(indexer++, new Operation() { 
          OperationCode = OperationCode.Ldstr, Value = notNullParameter.Name.Value });
        operations.Insert(indexer++, new Operation() { 
          OperationCode = OperationCode.Newobj, Value = this.argumentNullExceptionCtor });
        operations.Insert(indexer++, new Operation() { 
          OperationCode = OperationCode.Throw });
      }
    }
```

First I get all the parameters from the method that are marked as `[NotNull]` and that aren't value types. You can't specify in an attribute that it can only go on reference or value type-based parameters, and since a value type is always not null, they are ignored even if they have `[NotNull]` on them. Then I enumerate that list and add the not-null checking code for each parameter, which is always the same 5 opcodes. I get the `Operation` list from the method's `Body` property, and I inject those operations just before the first operation. Note the value that's used for the `Brtrue_S` operation, though. Cecil makes this code easier because you can reference an `Instruction` (`Operation` in CCI) when you inject a "break" `Instruction`; CCI doesn't seem to support this. That's where my `Label` class comes in handy - it ensures that I'll be "pointing to" what was the first operation before I did my injection.

It's easier if I just show you what the IL looks like before and after injection occurs. Here's the original IL stream:

```
.method public hidebysig specialname rtspecialname instance void .ctor(
  valuetype [mscorlib]System.Guid id, int32 age, string firstName, 
  string lastName, valuetype CciInjected.BirthDate birthDate) cil managed
{
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.TraceAttribute::.ctor()
  .param [2]
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.NotNullAttribute::.ctor()
  .param [3]
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.NotNullAttribute::.ctor()
  .param [4]
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.NotNullAttribute::.ctor()
  .maxstack 8
  L_0000: ldarg.0 
  L_0001: call instance void CciInjected.AttributedCustomer::.ctor()
  L_0006: nop 
  L_0007: nop 
  L_0008: ldarg.0 
  L_0009: ldarg.1 
  L_000a: call instance void [Customers]Customers.Customer::set_Id(
    valuetype [mscorlib]System.Guid)
  L_000f: nop 
  L_0010: ldarg.0 
  L_0011: ldarg.2 
  L_0012: call instance void [Customers]Customers.Customer::set_Age(int32)
  L_0017: nop 
  L_0018: ldarg.0 
  L_0019: ldarg.3 
  L_001a: call instance void [Customers]Customers.Customer::set_FirstName(string)
  L_001f: nop 
  L_0020: ldarg.0 
  L_0021: ldarg.s lastName
  L_0023: call instance void [Customers]Customers.Customer::set_LastName(string)
  L_0028: nop 
  L_0029: ldarg.0 
  L_002a: ldarg.s birthDate
  L_002c: call instance void CciInjected.AttributedCustomer::set_BirthDate(
    valuetype CciInjected.BirthDate)
  L_0031: nop 
  L_0032: nop 
  L_0033: ret 
}
```

Here's what it looks like after injection with the changes in bold:

```
.method public hidebysig specialname rtspecialname instance void .ctor(
  valuetype [mscorlib]System.Guid id, int32 age, string firstName, 
  string lastName, valuetype CciInjected.BirthDate birthDate) cil managed
{
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.TraceAttribute::.ctor()
  .param [2]
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.NotNullAttribute::.ctor()
  .param [3]
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.NotNullAttribute::.ctor()
  .param [4]
  .custom instance void 
    [CciInjector.Attributes]CciInjector.Attributes.NotNullAttribute::.ctor()
  .maxstack 8
  L_0000: ldarg lastName
  L_0004: brtrue.s L_0011
  L_0006: ldstr "lastName"
  L_000b: newobj instance void [mscorlib]System.ArgumentNullException::.ctor(string)
  L_0010: throw 
  L_0011: ldarg firstName
  L_0015: brtrue.s L_0022
  L_0017: ldstr "firstName"
  L_001c: newobj instance void [mscorlib]System.ArgumentNullException::.ctor(string)
  L_0021: throw
  L_0022: ldarg.0 
  L_0023: call instance void CciInjected.AttributedCustomer::.ctor()
  L_0028: nop 
  L_0029: nop 
  L_002a: ldarg.0 
  L_002b: ldarg.1 
  L_002c: call instance void [Customers]Customers.Customer::set_Id(
    valuetype [mscorlib]System.Guid)
  L_0031: nop 
  L_0032: ldarg.0 
  L_0033: ldarg.2 
  L_0034: call instance void [Customers]Customers.Customer::set_Age(int32)
  L_0039: nop 
  L_003a: ldarg.0 
  L_003b: ldarg.3 
  L_003c: call instance void [Customers]Customers.Customer::set_FirstName(string)
  L_0041: nop 
  L_0042: ldarg.0 
  L_0043: ldarg.s lastName
  L_0045: call instance void [Customers]Customers.Customer::set_LastName(string)
  L_004a: nop 
  L_004b: ldarg.0 
  L_004c: ldarg.s birthDate
  L_004e: call instance void CciInjected.AttributedCustomer::set_BirthDate(
    valuetype CciInjected.BirthDate)
  L_0053: nop 
  L_0054: nop 
  L_0055: ret 
}
```
You can see that the breaks end up at the right spot.

By the way, in case you're wondering what you should set `Value` to in an `Operation` based on the value of `OperationCode` (since `Value` is typed as an `object`), check out `SerializeMethodBodyIL()` in `ILGenerator` - it'll explain what it expects to see in that property.

In Part 5 I'll dive into the tracing injection and `ToString()` implementation.

> Published: 05.07.2009 07:55:21 AM CST