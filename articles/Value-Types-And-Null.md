---
title: Value Types and Null
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Value Types and Null

I just ran into this code that someone wrote in our code base:

```c#
public class Test
{
  private bool aValue;
    
  public bool AValue
  {
    get
    {
      if(this.aValue == null)
      {
        this.aValue = false;
      }
            
      return this.aValue;
    }
  }
}
```

Yeah ... that really doesn't look right. `aValue` is a value type, so it'll never be null. In fact, the compiler gives me two warnings with this code: "The result of the expression is always 'false' since a value of type 'bool' is never equal to 'null' of type 'bool?'" (the null test) and "Unreachable code detected" (the code within the if statement) [1]. But I was curious what the compiler created in the IL stream. My guess was a boxing operation was going on, so I opened up the assembly in Reflector and took a look at the byte codes:

```
.method public hidebysig specialname instance bool get_AValue() cil managed
{
  .maxstack 8
  L_0000: ldarg.0 
  L_0001: ldfld bool ValueTypesAndNull.Test::aValue
  L_0006: ret 
}
```

Haha - this is when the project is in Release mode. Let's change it to Debug:

```
.method public hidebysig specialname instance bool get_AValue() cil managed
{
  .maxstack 8
  L_0000: ldarg.0 
  L_0001: ldfld bool ValueTypesAndNull.Test::aValue
  L_0006: ret 
}
```

Nice - the compiler realizes that the code is pretty stupid in both modes and doesn't even bother handling that code. Note that in the Debug build, the pdb contains all of the sequence points for the code - I confirmed this using Mike Stall's excellent pdb2xml tool.

[1] This is a good reason to "treat warnings as errors".

> Published: 01.25.2007 08:24:45 AM CST