---
title: On-The-Fly Assembly Modification
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# On-The-Fly Assembly Modification

One of my favorite namespaces is `System.Reflection.Emit`. Lately I've been revisiting that namespace to upgrade a project I worked on (which is in my CIL book - it's the dynamic proxies code) and I've been in heaven again. However, one thing that's always bugged me is that .NET has never shipped with an API to read and write assemblies on the fly. A number of individuals and groups have tried to create assemby reader/writers and this past week I took a look at a couple of them - [Cecil](https://github.com/jbevain/cecil) and PERWAPI.

To make a long story short, I immediately fell in love with Cecil. Its object model is pretty easy to use. My first test with Cecil was to mimic that tutorial but to add pre- and post-invocation `Console.Out.WriteLine()` notifications along with notifications of an impending exception. Cecil made that very easy to do. Here's the main method to rip through all of the methods in all of the types within an assembly:

```c#
public void AddCallIndentifiers()
{
  AssemblyDefinition assembly = AssemblyFactory.GetAssembly("Modifications.dll");

  foreach (TypeDefinition type in assembly.MainModule.Types)
  {
    if (type.Name != "<Module>")
    {
      List<MethodDefinition> methods = new List<MethodDefinition>();
      MethodDefinition[] methodArray = new MethodDefinition[type.Methods.Count];
      type.Methods.CopyTo(methodArray, 0);
      methods.AddRange(methodArray); 
      methodArray = new MethodDefinition[type.Constructors.Count];
      type.Constructors.CopyTo(methodArray, 0);
      methods.AddRange(methodArray);

      foreach (MethodDefinition method in methods)
      {
        this.AddCalls(method);
      }

      assembly.MainModule.Import(type);
    }
  }

  AssemblyFactory.SaveAssembly(assembly, @"..\..\..\Modifications.dll");
}
```

The CIL injection is done in `AddCalls()`:

```c#
private void AddCalls(MethodDefinition method)
{
  AssemblyDefinition assembly = method.DeclaringType.Module.Assembly;
  VariableDefinition methodDescription = new VariableDefinition(
    assembly.MainModule.Import(typeof(string)));
  method.Body.Variables.Add(methodDescription);

  MethodReference writeLine = assembly.MainModule.Import(
    typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }));
  MethodReference concat = assembly.MainModule.Import(
    typeof(string).GetMethod("Concat", new Type[] { typeof(string), typeof(string) }));
  MethodReference getCurrentMethod = assembly.MainModule.Import(
    typeof(MethodBase).GetMethod("GetCurrentMethod", Type.EmptyTypes));
  MethodReference toString = assembly.MainModule.Import(
    typeof(MethodBase).GetMethod("ToString", Type.EmptyTypes));

  CilWorker worker = method.Body.CilWorker;

  // Store the method name.
  Instruction firstInstruction = method.Body.Instructions[0];
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Call, getCurrentMethod));
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Callvirt, toString));
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Stloc, methodDescription));

  // Emit the "started" trace.
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Ldloc, methodDescription));
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Ldstr, " started"));
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Call, concat));
  worker.InsertBefore(firstInstruction,
    worker.Create(OpCodes.Call, writeLine));

  List<Instruction> returns = new List<Instruction>();
  List<Instruction> throws = new List<Instruction>();

  foreach (Instruction i in method.Body.Instructions)
  {
    if (i.OpCode == OpCodes.Ret)
    {
      returns.Add(i);
    }
    else if (i.OpCode == OpCodes.Throw)
    {
      throws.Add(i);
    }
  }

  foreach (Instruction @return in returns)
  {
    worker.InsertBefore(@return,
      worker.Create(OpCodes.Ldloc, methodDescription));
    worker.InsertBefore(@return,
      worker.Create(OpCodes.Ldstr, " finished"));
    worker.InsertBefore(@return,
      worker.Create(OpCodes.Call, concat));
    worker.InsertBefore(@return,
      worker.Create(OpCodes.Call, writeLine));
  }

  foreach (Instruction @throw in throws)
  {
    worker.InsertBefore(@throw,
      worker.Create(OpCodes.Ldloc, methodDescription));
    worker.InsertBefore(@throw,
      worker.Create(OpCodes.Ldstr, " finished with exception"));
    worker.InsertBefore(@throw,
      worker.Create(OpCodes.Call, concat));
    worker.InsertBefore(@throw,
      worker.Create(OpCodes.Call, writeLine));
  }
}
```

By the way, here's what the `ModifyMe` class looks like in Modifications.dll:

```c#
public sealed class ModifyMe
{
  public T ReflectValue<T>(T value)
  {
    return value;
  }

  public int ReflectValue(int value)
  {
    return value;
  }

  private bool IsEven(int x)
  {
    return x % 2 == 0;
  }

  public void VeryBadMethod()
  {
    int x = 3;

    for (int i = 0; i < 5; i++)
    {
      if (i == x)
      {
        throw new NotSupportedException();
      }
    }
  }

  public int WhackValue(int x)
  {
    if (this.IsEven(x))
    {
      return x / 2;
    }
    else
    {
      if (x == 3)
      {
        return 5;
      }
      else
      {
        return -1;
      }
    }
  }
}
```

Now, when I substitute the Modifications.dll into a directory where a console application uses the injected code ...

```c#
static void Main(string[] args)
{
  MethodBase current = MethodBase.GetCurrentMethod();
  Console.Out.WriteLine(current.ToString());

  ModifyMe modMe = new ModifyMe();
  Console.Out.WriteLine(modMe.ReflectValue(3));
  Console.Out.WriteLine(modMe.WhackValue(22));

  try
  {
    modMe.VeryBadMethod();
  }
  catch (NotSupportedException)
  {
    Console.Out.WriteLine("I caught it.");
  }
}
```

the output looks like this:

```
Void Main(System.String[])
Void .ctor() started
Void .ctor() finished
Int32 ReflectValue(Int32) started
Int32 ReflectValue(Int32) finished
3
Int32 WhackValue(Int32) started
Boolean IsEven(Int32) started
Boolean IsEven(Int32) finished
Int32 WhackValue(Int32) finished
11
Void VeryBadMethod() started
Void VeryBadMethod() finished with exception
I caught it.
Press any key to continue . . .
```

That is so sweet! This is a pretty simple example, yet the results set my mind to think about all sorts of cool, cool things I could do with this.

I tried to mimic this code with PERWAPI, and I got stuck in a hurry. I couldn't get a `MethodRef` for `Console.Out.WriteLine()`, and digging through the source code it looks like it only loads up a certain amount of types from mscorlib.dll. I could be wrong on this, but I had immediate success with Cecil so I'm sticking with that API.

My personal wish is that .NET would add an API to allow developers to read/write assemblies on the fly. They're pretty close right now, but there's no way (at least not easily) to open an assembly, muck with the type definitions and method implementations and save the results [1]. Maybe in the future they'll do this ... one can always hope!

[1] I should note that the assembly modification process has to occur before the console application is executed. Otherwise the console locks up the assembly and I can't save the results. I'm planning on doing an interesting (at least I think it's interesting) application with Cecil shortly so I'm going to look at ways to make this assembly modification process less painful - i.e. avoid the assembly locking issue as much as I can with very little pain.

> Published: 10.25.2006 06:51:43 PM CST