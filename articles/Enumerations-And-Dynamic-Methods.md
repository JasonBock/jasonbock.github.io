---
title: Enumerations and Dynamic Methods
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Enumerations and Dynamic Methods

Enumerations are tricky little beasts. They look so simplistic:

```c#
public enum IntEnumeration
{
  ValueOne = 12,
  ValueTwo = 32,
  ValueThree = 44444
}
```

but to actually manipulate them can be hair-pulling (go to this article for some other wacky enumeration behaviors) . Check out this code:

```c#
foreach(object intValue in Enum.GetValues(typeof(IntEnumeration)))
{
  Console.Out.WriteLine(Program.OutputFormat, 
    "foreach", intValue, intValue.GetType().FullName);
}
```

Here's what gets printed to the console window:

```
foreach - ValueOne - UnderlyingEnumerationType.IntEnumeration
foreach - ValueTwo - UnderlyingEnumerationType.IntEnumeration
foreach - ValueThree - UnderlyingEnumerationType.IntEnumeration
```

When I ran this the first time, the results were not what I was expecting. I thought I'd get an array of values that were cast to the underlying type of the enumeration, not an array of values that are typed as the enumeration itself. The `Enum` class has a method called `GetUnderlyingType()`, but unfortunately that just tells us the type of enumeration; we can't use it to cast values.

Or can we?

I saw an (unrelated) article about anonymous types in LINQ, and it shows a `Cast()` method that I thought, well, couldn't I use it to cast an enumeration value? So I started writing a little framework:

```c#
public abstract class EnumUtility
{
  public EnumUtility() : base()
  {
  }
    
  public object Convert(object enumeration)
  {
    if(!enumeration.GetType().IsEnum)
    {
      throw new ArgumentException(
        "The given value is not an enumeration.", "enumeration");
    }

    return this.OnConvert(enumeration);
  }

  protected abstract object OnConvert(object enumeration);
}

public sealed class CastEnumUtility : EnumUtility
{
  protected override object OnConvert(object enumeration)
  {
    return Cast(enumeration, 
      Activator.CreateInstance(Enum.GetUnderlyingType(enumeration.GetType())));	
  }

  T Cast<T>(object obj, T type)
  {
    return (T)obj;
  }
}
```

Unfortunately, that doesn't help much:

```c#
EnumUtility castEnum = new CastEnumUtility();

foreach(object intValue in Enum.GetValues(typeof(IntEnumeration)))
{
  Console.Out.WriteLine(Program.OutputFormat, 
    "foreach", intValue, intValue.GetType().FullName);
  object castValue = castEnum.Convert(intValue);
  Console.Out.WriteLine(Program.OutputFormat,
    "Cast", castValue, castValue.GetType().FullName);
}
```

Here's what gets printed to the console window with this adjustment:

```
foreach - ValueOne - UnderlyingEnumerationType.IntEnumeration
Cast - ValueOne - UnderlyingEnumerationType.IntEnumeration
foreach - ValueTwo - UnderlyingEnumerationType.IntEnumeration
Cast - ValueTwo - UnderlyingEnumerationType.IntEnumeration
foreach - ValueThree - UnderlyingEnumerationType.IntEnumeration
Cast - ValueThree - UnderlyingEnumerationType.IntEnumeration
```

But with a little `DynamicMethod` love, we can accomplish our goal:

```c#
internal delegate object ConvertEnumDelegate(object enumeration);

public sealed class DynamicMethodEnumUtility : EnumUtility
{
  private Dictionary<Type, ConvertEnumDelegate> converters =
    new Dictionary<Type, ConvertEnumDelegate>();

  public DynamicMethodEnumUtility()
    : base()
  {
    this.CreateConverter(typeof(int));
  }

  protected override object OnConvert(object enumeration)
  {
    ConvertEnumDelegate converter = null;

    Type underlyingType = Enum.GetUnderlyingType(enumeration.GetType());

    if(!this.converters.ContainsKey(underlyingType))
    {
      converter = this.CreateConverter(underlyingType);
      this.converters.Add(underlyingType, converter);
    }
    else
    {
      converter = this.converters[underlyingType];
    }

    return converter(enumeration);
  }

  private ConvertEnumDelegate CreateConverter(Type underlyingType)
  {
    Type[] arguments = { typeof(object) };
    DynamicMethod converter = new DynamicMethod("ConvertTo" + underlyingType.Name,
      typeof(object), arguments, typeof(DynamicMethodEnumUtility).Module);

    ILGenerator generator = converter.GetILGenerator();
    generator.Emit(OpCodes.Ldarg_0);
    generator.Emit(OpCodes.Unbox_Any, underlyingType);
    generator.Emit(OpCodes.Box, underlyingType);
    generator.Emit(OpCodes.Ret);

    return (ConvertEnumDelegate)converter.CreateDelegate(typeof(ConvertEnumDelegate));
  }
}
```

Now I finally get the output I want!

```c#
EnumUtility castEnum = new CastEnumUtility();
EnumUtility dynamicEnum = new DynamicMethodEnumUtility();

foreach(object intValue in Enum.GetValues(typeof(IntEnumeration)))
{
  Console.Out.WriteLine(Program.OutputFormat, 
    "foreach", intValue, intValue.GetType().FullName);
  object castValue = castEnum.Convert(intValue);
  Console.Out.WriteLine(Program.OutputFormat,
    "Cast", castValue, castValue.GetType().FullName);
  object dynamicValue = dynamicEnum.Convert(intValue);
  Console.Out.WriteLine(Program.OutputFormat,
    "Dynamic", dynamicValue, dynamicValue.GetType().FullName);
}
```

Here's what gets printed to the console window with this final adjustment:

```
foreach - ValueOne - UnderlyingEnumerationType.IntEnumeration
Cast - ValueOne - UnderlyingEnumerationType.IntEnumeration
Dynamic - 12 - System.Int32
foreach - ValueTwo - UnderlyingEnumerationType.IntEnumeration
Cast - ValueTwo - UnderlyingEnumerationType.IntEnumeration
Dynamic - 32 - System.Int32
foreach - ValueThree - UnderlyingEnumerationType.IntEnumeration
Cast - ValueThree - UnderlyingEnumerationType.IntEnumeration
Dynamic - 44444 - System.Int32
```

I'm not saying this the right way to do this. Nor am I saying that it's performant (there's some boxing and unboxing going on that I'm not too thrilled about). But it does accomplish what I wanted, and all it took was a little dynamic shim method. Very nice!

> Published: 03.26.2007 12:35:08 PM CST