---
title: Attaching Dynamic Methods
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Attaching Dynamic Methods

[Yesterday](https://jasonbock.net/articles/Enumerations-And-Dynamic-Methods) I did a bit of working using `DynamicMethod`. I have to admit, that was the first time I touched that class. I like diving into the Emit namespace, but I haven't had the need to crank out a shim method like that. Anyway, as I was reading the documentation, I found this:

> If the dynamic method is associated with a type, it has access to the private members of the type.

Just what does that mean, exactly? It smelled like something that I've seen in JavaScript:

```javascript
function State(value)
{
  this.__value__ = value;
}

State.prototype.GetValue = function()
{
  return this.__value__;
};

var s = new State(10);
WScript.Echo(s.GetValue());
WScript.Echo(s.__value__);

try
{
  s.SetValue(20);
}
catch(e)
{
  WScript.Echo(e.message);
}

State.prototype.SetValue = function(value)
{
  this.__value__ = value;
};

s.SetValue(20);
WScript.Echo(s.GetValue());
```

This shows how calling a function `SetValue` doesn't work until the prototype is set up. Running this under wscript gives this:

```
10
10
Object doesn't support this property or method
20
```

This isn't anything new for developers who use dynamic languages (ala Python, Ruby, etc.). But I was curious to see if `DynamicMethod` gives similar behavior:

```c#
internal static class Program
{
  private delegate void SetValueClassDelegate(State state, int value);
  private delegate void SetValueObjectDelegate(int value);
    
  internal static void Main(string[] args)
  {
    State state = new State(10);
    Console.Out.WriteLine(state.Value);
    SetValueObjectDelegate setValueObject = Program.AttachMethodToObject(state);
    setValueObject(20);
    Console.Out.WriteLine(state.Value);
            
    State state2 = new State(100);
    Console.Out.WriteLine(state2.Value);
    setValueObject(200);
    Console.Out.WriteLine(state2.Value);

    SetValueClassDelegate setValueClass = Program.AttachMethodToClass();
    State state3 = new State(1000);
    Console.Out.WriteLine(state3.Value);
    setValueClass(state3, 2000);
    Console.Out.WriteLine(state3.Value);
  }

  private static SetValueObjectDelegate AttachMethodToObject(State state)
  {
    Type[] arguments = { typeof(State), typeof(int) };
    DynamicMethod converter = new DynamicMethod("SetValueObject",
      null, arguments, typeof(State));

    ILGenerator generator = converter.GetILGenerator();
    generator.Emit(OpCodes.Ldarg_0);
    generator.Emit(OpCodes.Ldarg_1);
    generator.Emit(OpCodes.Stfld, 
      typeof(State).GetField("value", BindingFlags.NonPublic | BindingFlags.Instance));
    generator.Emit(OpCodes.Ret);

    return (SetValueObjectDelegate)converter.CreateDelegate(typeof(SetValueObjectDelegate), state);
  }

  private static SetValueClassDelegate AttachMethodToClass()
  {
    Type[] arguments = { typeof(State), typeof(int) };
    DynamicMethod converter = new DynamicMethod("SetValueClass",
      null, arguments, typeof(State));

    ILGenerator generator = converter.GetILGenerator();
    generator.Emit(OpCodes.Ldarg_0);
    generator.Emit(OpCodes.Ldarg_1);
    generator.Emit(OpCodes.Stfld,
      typeof(State).GetField("value", BindingFlags.NonPublic | BindingFlags.Instance));
    generator.Emit(OpCodes.Ret);

    return (SetValueClassDelegate)converter.CreateDelegate(typeof(SetValueClassDelegate));
  }
}
```

Where `State` looks like this:

```c#
internal sealed class State
{
  private int value;
    
  internal State() : base()
  {
  }
    
  internal State(int value) : this()
  {
    this.value = value;
  }
    
  internal int Value
  {
    get
    {
      return this.value;
    }
  }
}
```

Running this code gives the following:

```
10
20
100
100
1000
2000
```

What this shows to me is that you can add new methods to specific object instances (you can also do this in JavaScript - I just didn't show that in my code example). However, you can't attach a method to a class such that it's a part of every class instance (for some reason I think a Smalltalker is telling me, "if only a class was an object ... "). You can attach a method to a type - in this case, it acts like a static method. In either case, both methods have access to the internals of the object. I tried using `InvokeMethod()` to see if I could "find" the new method, and that doesn't seem to work (although my code could be wrong too).

Dynamic methods are another cool aspect of .NET. Just do a search on "DynamicMethod" and you'll find all sorts of cool little nuggests out there. If your code has the right permissions, you're never limited by your language; you can always get around limitations by visiting the Emit namespace.

> Published: 03.27.2007 08:30:24 PM CST