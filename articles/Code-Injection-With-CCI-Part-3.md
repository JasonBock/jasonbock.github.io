---
title: Code Injection With CCI - Part 3
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 3

Previous Parts

* [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html)
* [Part 2](https://jasonbock/articles/Code-Injection-With-CCI-Part-2.html)
* [Part 2.5](https://jasonbock/articles/Code-Injection-With-CCI-Part-25.html)

In Part 2, I covered the shell of `CciInjector`, which included looking at how assemblies are loaded and the sent to the visitor class. In this part, I'm going to cover the fixup routines.

OK, I know I originally said in Part 2 that I was going to cover how `[NotNull]` was handled. But as I started to actually implement more of the modifiers, I quickly ran into a big snag: branches. The problem is as I'm modifying IL, I'm potentially (and more often than not, probably) breaking the offsets for the branch opcodes, which will quickly lead to nasty, ugly exceptions and/or unexpected results during method execution. In Cecil, I didn't have to worry about this as it automatically fixed all this up for you, but not so in CCI [1].

So I had to add this in myself. First, I created a `Label` class:

```c#
internal sealed class Label
{
  internal Label(IOperation operation)
    : base()
  {
    this.Operation = operation;
  }
    
  internal IOperation Operation
  {
    get;
    private set;
  }
}
```

The only thing it does is keeps a reference to an `IOperation`-based object.

Next, I look for branch opcodes in the operation stream, and change the value of the `Value` property with a `Label` object that "points to" the operation it will branch to. I do this before I do any method modifications so I can capture where the branches should go. Then once my visitations are done, I'll revisit the list and remove the labels with the correct offsets.

Here's the first part of the process where I inject all the labels into the exisiting IL stream:

```c#
private void TrackBranches(MethodDefinition method)
{
  var branchOperations = from operation in (method.Body as MethodBody).Operations
    where operation.OperationCode.IsBranch()
    select operation;

  foreach(var branchOperation in branchOperations)
  {
    // TODO: have to handle Switch differently.
    (branchOperation as Operation).Value = new Label(
      (from operation in method.Body.Operations
      where operation.Offset == (uint)branchOperation.Value
      select operation).First());
  }

  this.exceptionMappings = new Dictionary<uint, IOperation>();
  var handlerOffsets = new HashSet<uint>();
    
  foreach(var exceptionInformation in method.Body.OperationExceptionInformation)
  {
    handlerOffsets.Add(exceptionInformation.FilterDecisionStartOffset);
    handlerOffsets.Add(exceptionInformation.HandlerEndOffset);
    handlerOffsets.Add(exceptionInformation.HandlerStartOffset);
    handlerOffsets.Add(exceptionInformation.TryEndOffset);
    handlerOffsets.Add(exceptionInformation.TryStartOffset);
  }

  this.AddExceptionOperationMappings(handlerOffsets, method.Body.Operations);
}
```

First, I get all of the operations that use a "branch" opcode. This is determined by the `IsBranch()` extension method:

```c#
internal static bool IsBranch(this OperationCode @this)
{
  return @this == OperationCode.Beq || @this == OperationCode.Beq_S ||
    @this == OperationCode.Bge || @this == OperationCode.Bge_S ||
    @this == OperationCode.Bge_Un || @this == OperationCode.Bge_Un_S ||
    @this == OperationCode.Bgt || @this == OperationCode.Bgt_S ||
    @this == OperationCode.Bgt_Un || @this == OperationCode.Bgt_Un_S ||
    @this == OperationCode.Ble || @this == OperationCode.Ble_S ||
    @this == OperationCode.Ble_Un || @this == OperationCode.Ble_Un_S ||
    @this == OperationCode.Blt || @this == OperationCode.Blt_S ||
    @this == OperationCode.Blt_Un || @this == OperationCode.Blt_Un_S ||
    @this == OperationCode.Bne_Un || @this == OperationCode.Bne_Un_S ||
    @this == OperationCode.Br || @this == OperationCode.Br_S ||
    @this == OperationCode.Break || 
    @this == OperationCode.Brfalse || @this == OperationCode.Brfalse_S ||
    @this == OperationCode.Brtrue || @this == OperationCode.Brtrue_S ||
    @this == OperationCode.Cgt || @this == OperationCode.Cgt_Un ||
    @this == OperationCode.Clt || @this == OperationCode.Clt_Un ||
    @this == OperationCode.Jmp ||
    @this == OperationCode.Leave || @this == OperationCode.Leave_S ||
    @this == OperationCode.Switch;
}
```

Then, for each branch operation I find, I change its value to a `Label` which refers to the operation that the branch statement is going to jump to. Note my TODO comment in the code. I'm currently not handling switch opcodes right now - it's not that it would be hard to handle the jump table, I just haven't had the time to create a test example that generates a switch so I know I'm handling it correctly.

Finally, I handle exception handlers. Exception handler information is stored in an `IOperationExceptionInformation`-based object, and what we need to grab are all of the offset values from each handler. Due to the way handlers are specified, the values are usually duplicated, which is why I use a `HashSet`. Here's how the operations are tracked for exceptions:

```c#
private void AddExceptionOperationMappings(HashSet<uint> offsets, IEnumerable<IOperation> operations)
{
  foreach(var offset in offsets)
  {
    if(!this.exceptionMappings.ContainsKey(offset))
    {
      this.exceptionMappings.Add(offset,
        (from operation in operations
        where operation.Offset == offset
        select operation).First());
    }
  }
}
```

Once I'm done with all of my modifications, I call `FixBranches()` to reset all the offsets and strip out the `Label`s:

```c#
private void FixBranches(MethodDefinition method)
{
  var offset = 0u;

  foreach(var operation in (method.Body as MethodBody).Operations)
  {
    (operation as Operation).Offset = offset;
    offset += operation.OperationCode.Size(
      (operation.Value != null && operation.Value is uint[] ? 
        ((uint[])operation.Value).Length : 0));
  }

  var branchOperations = from operation in (method.Body as MethodBody).Operations
    where operation.OperationCode.IsBranch()
    select operation;

  foreach(var branchOperation in branchOperations)
  {
    (branchOperation as Operation).Value = (branchOperation.Value as Label).Operation.Offset;
  }

  foreach(var item in (method.Body as MethodBody).OperationExceptionInformation)
  {
    var exceptionOperation = item as OperationExceptionInformation;
    exceptionOperation.FilterDecisionStartOffset = 
      this.exceptionMappings[exceptionOperation.FilterDecisionStartOffset].Offset;
    exceptionOperation.HandlerEndOffset = 
      this.exceptionMappings[exceptionOperation.HandlerEndOffset].Offset;
    exceptionOperation.HandlerStartOffset = 
      this.exceptionMappings[exceptionOperation.HandlerStartOffset].Offset;
    exceptionOperation.TryEndOffset = 
      this.exceptionMappings[exceptionOperation.TryEndOffset].Offset;
    exceptionOperation.TryStartOffset = 
      this.exceptionMappings[exceptionOperation.TryStartOffset].Offset;
  }
}
```

`Size()` is another extension method for `OperationCode` - it's quite lengthy so I won't show it here, but basically it figures out the size of the current `Operation` as it would end up in the IL stream. I use that value to determine the new offset value for the current operation stream and change the values of the operations for the current value of offset. Once I fixed all the offsets, I remove all the labels. I do this by looking for all the branch operations, get the `Value` property as a `Label` for each operation, and change the operation's `Value` property to offset value that the `Label` is pointing at. Finally, I go through all the exception handlers and change their offset values to the corrected values.

Before I finish, here's how I'm using these methods in my custom `MetadataMutator` class called `InjectorVisitor`:

```c#
internal sealed class InjectorVisitor : MetadataMutator
{
  public override MethodDefinition Visit(MethodDefinition methodDefinition)
  {
    var method = base.Visit(methodDefinition);

    this.TrackBranches(method);
    // Do the modifications here ...
    this.FixBranches(method);

    return method;
  }
}
```

This puts all the fixup handling in one spot. The only thing I have to keep in mind is when I inject any branch-based opcode during my modifications, I have to set the `Value` property to a `Label` and not an offset value; otherwise `FixBranches()` will error out when it tries to cast `Value` as a `Label`.

In Part 4, I'll finally get around to showing how I support `[NotNull]`.

[1] There is an `ILGenerator` class that has fixup capabilities in CCI. In a reply to one of my posts Herman (a CCI developer) hinted at how `ILGenerator` could be used. It's definitely an avenue I'm going to pursue in the future, but I'm going to stay the course with this series of posts and take a look at the ideas he mentioned in the future.

> Published: 05.01.2009 11:54:39 AM CST