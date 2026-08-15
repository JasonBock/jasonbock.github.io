---
title: Code Injection With CCI - Part 5
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 5

Previous Parts

* [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html)
* [Part 2](https://jasonbock/articles/Code-Injection-With-CCI-Part-2.html)
* [Part 2.5](https://jasonbock/articles/Code-Injection-With-CCI-Part-25.html)
* [Part 3](https://jasonbock/articles/Code-Injection-With-CCI-Part-3.html)
* [Part 4](https://jasonbock/articles/Code-Injection-With-CCI-Part-4.html)

In this part, I'll wrap up the implementation of the injectors by looking at how I add a `ToString()` implementation to a class and how I add tracing capabilities to each method invocation.

Now that we have a lot of the heavy lifting done, it's pretty straightforward to add in these two last features. First, let's add our two method calls that handle these new injections to the right `Visit()` methods:

```c#
public override MethodDefinition Visit(MethodDefinition methodDefinition)
{
  var method = base.Visit(methodDefinition);

  this.TrackBranches(method);
  this.AddNotNullChecks(method);
  this.AddTraceCalls(method);
  this.FixBranches(method);

  return method;
}

protected override void Visit(TypeDefinition typeDefinition)
{
  base.Visit(typeDefinition);
  this.AddToString(typeDefinition);
}
```

Let's dive into `AddTraceCalls()` first. It's kind of a long method as it stands right now, so let's take it in pieces:

```c#
private void AddTraceCalls(MethodDefinition method)
{
  if(method.Attributes.Count > 0 &&
    AttributeHelper.Contains(method.Attributes, this.traceAttribute))
  {
    var body = method.Body as MethodBody;
    body.LocalsAreZeroed = true;
    var methodDescription = new LocalDefinition() { Type = this.host.PlatformType.SystemString };
    body.LocalVariables.Add(methodDescription);

    var writeLine = TypeHelper.GetMethod(
      this.mscorlib.GetType("System", "Console").ResolvedType,
      this.host.NameTable.GetNameFor("WriteLine"),
      this.host.PlatformType.SystemString);
    var concat = TypeHelper.GetMethod(
      this.host.PlatformType.SystemString.ResolvedType,
      this.host.NameTable.GetNameFor("Concat"),
      this.host.PlatformType.SystemString, this.host.PlatformType.SystemString);
    var getCurrentMethod = TypeHelper.GetMethod(
      this.mscorlib.GetType("System", "Reflection", "MethodBase").ResolvedType,
      this.host.NameTable.GetNameFor("GetCurrentMethod"));
    var toString = TypeHelper.GetMethod(
      this.host.PlatformType.SystemObject.ResolvedType,
      this.host.NameTable.GetNameFor("ToString"));
```

After I confirm that the method has been marked with `[Trace]`, I add a local variable that will hold the method name. Note that I have to set `LocalsAreZeroed` to `true`, otherwise I'll get a peverify error. I also get references to a bunch of methods that I'll use for tracing purposes. Once that's done, I add the first tracing code:

```c#
    var operations = body.Operations;
    var indexer = 0;

    operations.Insert(indexer++, new Operation() { 
      OperationCode = OperationCode.Call, Value = getCurrentMethod });
    operations.Insert(indexer++, new Operation() { 
      OperationCode = OperationCode.Callvirt, Value = toString });
    operations.Insert(indexer++, new Operation() { 
      OperationCode = OperationCode.Stloc, Value = methodDescription });

    InjectorVisitor.InsertTraceOperations(methodDescription, writeLine,
      concat, operations, indexer, " started");
```

`InsertTraceOperations()` contains code that I use a lot in this method to add pre, post, and exception point tracing. Let's take a look at that:

```c#
private static void InsertTraceOperations(LocalDefinition methodDescription, IMethodDefinition writeLine,
  IMethodDefinition concat, List<IOperation> operations, int indexer, string status)
{
  operations.Insert(indexer++, new Operation() { 
    OperationCode = OperationCode.Ldloc, Value = methodDescription });
  operations.Insert(indexer++, new Operation() { 
    OperationCode = OperationCode.Ldstr, Value = status });
  operations.Insert(indexer++, new Operation() { 
    OperationCode = OperationCode.Call, Value = concat });
  operations.Insert(indexer++, new Operation() { 
    OperationCode = OperationCode.Call, Value = writeLine });
}
```

Now that I have the method invocation traced (i.e. the "started" part), I can add the "finished" and "exception was thrown" parts. To do this, I look for `Ret`, `Throw`, and `Rethrow` operations:

```c#
    var returnOperations = from operation in operations
      where operation.OperationCode == OperationCode.Ret
      orderby operation.Offset
      select operation;
    var throwOperations = from operation in operations
      where (operation.OperationCode == OperationCode.Throw ||
        operation.OperationCode == OperationCode.Rethrow)
      orderby operation.Offset
      select operation;

    foreach(var returnOperation in returnOperations)
    {
      InjectorVisitor.InsertTraceOperations(methodDescription, writeLine,
        concat, operations, operations.IndexOf(returnOperation), " finished");
    }

    foreach(var throwOperation in throwOperations)
    {
      InjectorVisitor.InsertTraceOperations(methodDescription, writeLine,
        concat, operations, operations.IndexOf(throwOperation), " - exception was thrown");
    }
  }
}
```

And that's it! Now I'll have simple tracing for my methods.

The other injection is adding a `ToString()` implementation to the given class. This one is long too, so let's break it up into chunks:

```c#
private void AddToString(TypeDefinition type)
{
  if(type.Attributes.Count > 0 &&
    AttributeHelper.Contains(type.Attributes, this.toStringAttribute))
  {
    var method = TypeHelper.GetMethod(type,
      this.host.NameTable.GetNameFor("ToString"));

    if(method == null || method == Dummy.Method)
    {
      var toString = new MethodDefinition()
      {
        Body = new MethodBody(), CallingConvention = CallingConvention.HasThis,
        ContainingType = type, IsVirtual = true, IsHiddenBySignature = true,
        Name = this.host.NameTable.GetNameFor("ToString"),
        Type = this.host.PlatformType.SystemString,
        Visibility = TypeMemberVisibility.Public
      };

      var body = (toString.Body as MethodBody);
      body.MethodDefinition = toString;
      body.MaxStack = 8;
      var operations = body.Operations;

      var builder = this.mscorlib.GetType("System", "Text", "StringBuilder").ResolvedType;
      var builderCtor = TypeHelper.GetMethod(builder,
        this.host.NameTable.GetNameFor(".ctor"));
      var builderAppendObject = TypeHelper.GetMethod(builder,
        this.host.NameTable.GetNameFor("Append"),
        this.host.PlatformType.SystemObject);
      var builderAppendString = TypeHelper.GetMethod(builder,
        this.host.NameTable.GetNameFor("Append"),
        this.host.PlatformType.SystemString);
      var builderToString = TypeHelper.GetMethod(builder,
        this.host.NameTable.GetNameFor("ToString"));
```

Not only do I check to see if `[ToString]` is on the class, I also check to see if a `ToString()` method already exists. If it does, I don't overwrite what's there. But if it's not, I get a reference to a `StringBuilder` and the appropriate `Append()` and `ToString()` calls that I'll use a lot to create my string:

```c#
      var properties = (from property in this.GetProperties(type)
        where property.Getter != null
        where (property.Getter.ResolvedMethod.Visibility & 
          TypeMemberVisibility.Public) == TypeMemberVisibility.Public
        select property).ToList();

      if(properties.Count > 0)
      {
        operations.Add(new Operation() { 
          OperationCode = OperationCode.Newobj, Value = builderCtor });

        for(var i = 0; i < properties.Count; i++)
        {
          var property = properties[i];
          operations.Add(new Operation() { 
            OperationCode = OperationCode.Ldstr, Value = property.Name.Value + ": " });
          operations.Add(new Operation() { 
            OperationCode = OperationCode.Call, Value = builderAppendString });
          operations.Add(new Operation() { 
            OperationCode = OperationCode.Ldarg_0 });
          operations.Add(new Operation()
          {
            OperationCode = (property.Getter.ResolvedMethod.IsVirtual ? 
              OperationCode.Callvirt : OperationCode.Call),
            Value = property.Getter.ResolvedMethod
          });

          if(property.Type.IsValueType)
          {
            operations.Add(new Operation() { 
              OperationCode = OperationCode.Box, Value = property.Type.ResolvedType });
          }

          operations.Add(new Operation() { 
            OperationCode = OperationCode.Call, Value = builderAppendObject });

          if(i < properties.Count - 1)
          {
            operations.Add(new Operation() { 
              OperationCode = OperationCode.Ldstr, Value = " || " });
            operations.Add(new Operation() { 
              OperationCode = OperationCode.Call, Value = builderAppendString });
          }
        }

        operations.Add(new Operation() { 
          OperationCode = OperationCode.Callvirt, Value = builderToString });
      }
      else
      {
        operations.Add(new Operation() { 
          OperationCode = OperationCode.Ldstr, Value = string.Empty });
      }

      operations.Add(new Operation() { 
        OperationCode = OperationCode.Ret });
      type.Methods.Add(toString);
    }
  }
}
```

I first find all the public properties that are readable. If there are any, I create my `StringBuilder`; otherwise, I'll end up pushing an empty string onto the stack. Then I rip through that property list, and create my string of property name-value pairs. Basically, what I'm doing is getting the name of the property and its value and adding it to the `StringBuilder`, so I end up with something like this:

```
"property1: property1Value || property2: property2Value"
```

Note that if the property's type is a value type, I handle it a bit differently by boxing the value first. Once all the properties have been handled, I call `ToString()` on the `StringBuilder`, push that onto the stack, and return. And that's all there is to it!

In Part 6 (the final article in the series), I'll talk about some future work I'd like to do and my thoughts on CCI in general.

> Published: 05.21.2009 06:30:58 PM CST