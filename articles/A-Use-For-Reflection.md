---
title: A Use for Reflection
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Use for Reflection

I'm working on a new [Reflector](https://www.red-gate.com/products/reflector/) add-in (just something I'm entertaining at the moment - I have no idea if it'll really work or not, and I don't know if it'll have any value ... I'll keep y'all posted) and there's a part in the code where I have to walk the `IInstruction` list from an `IMethodBody` reference. `IInstruction` has a `Code` property, which corresponds to the opcode, but it's defined as an `int`. What I needed was a way to map that value to the IL instruction like "throw" or "ldstr". There's an `OpCode` structure in `System.Reflection.Emit` that's used on the `OpCodes` class, which contains all of the IL instructions as `OpCode`-typed fields, but I couldn't find a way to say, here's an IL instruction as a numeric value, please translate it into an `OpCode` for me. So what I decided to do was use a little mapping routine, like so:

```c#
private static Dictionary<short, OpCode> GetCodeMappings()
{
  Dictionary<short, OpCode> codeMappings = new Dictionary<short, OpCode>();

  foreach(FieldInfo codeField in typeof(OpCodes).GetFields())
  {
    if(typeof(OpCode).IsAssignableFrom(codeField.FieldType))
    {
      OpCode code = (OpCode)codeField.GetValue(null);
      codeMappings.Add(code.Value, code);
    }
  }

  return codeMappings;
}
```

This makes it really easy to find out if an `IInstruction` is a "ldstr" `OpCode` (assume `codeMappings` is a static field that was assigned the return value from `GetCodeMappings()`):

```c#
OpCode op = MyClass.codeMappings[(short)instruction.Code];

if(op == OpCodes.Ldstr)
{
  // do something...
}
```

I realize that I'm doing a cast from an `int` to a `short`, which isn't desirable. I'm not sure why `IInstruction` has `Code` defined as an `int` since `OpCode` uses a `short`, and I believe all opcodes are two bytes in length, so I've decided to use `short` as the source of truth.

Reflection is a handy tool to have around!

> Published: 11.29.2007 07:52:38 AM CST