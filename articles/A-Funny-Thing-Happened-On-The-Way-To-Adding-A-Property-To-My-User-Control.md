---
title: A Funny Thing Happened on the Way to Adding a Property to My User Control ...
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Funny Thing Happened on the Way to Adding a Property to My User Control ...

One day, not too long ago, I was working on a custom user control. It had gone through many iterations and revisions, but it was slowly coming together. On that fateful day, I had decided to add a class that would end up as a property on my `DrumSet` class. It was called ... `DrumSetAspects`, and it looked something like this:

```c#
public sealed class DrumSetAspects
{
  private bool hasDoubleBassPedals = false;
  private bool hasGong = false;
  private bool hasElectronicDrums = false;

  public DrumSetAspects() : base() {}

  public DrumSetAspects(bool hasDoubleBassPedals, 
    bool hasElectronicDrums, bool hasGong) : this()
  {
    this.hasDoubleBassPedals = hasDoubleBassPedals;
    this.hasGong = hasGong;
    this.hasElectronicDrums = hasElectronicDrums;
  }

  public bool HasDoubleBassPedals
  {
    get
    {
      return this.hasDoubleBassPedals;
    }
    set
    {
      this.hasDoubleBassPedals = value;
    }
  }

  public bool HasElectronicDrums
  {
    get
    {
      return this.hasElectronicDrums;
    }
    set
    {
      this.hasElectronicDrums = value;
    }
  }

  public bool HasGong
  {
    get
    {
      return this.hasGong;
    }
    set
    {
      this.hasGong = value;
    }
  }
}
```

The property on `DrumSet` was trivial:

```c#
public class DrumSet : System.Windows.Forms.UserControl
{
  private DrumSetAspects aspects = new DrumSetAspects();

  public DrumSetAspects Aspects
  {
    get
    {
      return this.aspects;
    }
    set
    {
      if(value != null)
      {
        this.aspects = value;
      }
    }
  }
}
```

Now I dropped an instance of `DrumSet` on to a `Band` user control. However, I was very suprised when I saw my `Aspects` property in the *Properties* window:

![UserControl Property](https://jasonbock.net/images/UserControl-Property-1.png "UserControl Property")

What?? I can't change the value of the `Aspects` property like I can with the `Size` property?! I mean, look at this:

![UserControl Property](https://jasonbock.net/images/UserControl-Property-2.png "UserControl Property")

See? I can change individual property values (`Height` and `Width`) and I can also change the entire value of `Size` by changing the "56, 80" string value. Why doesn't VS .NET just automagically do its Reflectification on `DrumSetAspects` and give me this tree view on my `Aspects` property?

Well, I quickly realized there must be some attribute goodness on Size that doesn't exist on `DrumSetAspects`. So, I took a breath, and took a look at the `Size` property on `Control` in the .NET SDK:

```c#
public Size Size {get; set;}
```

Crud - no metadata information there. I then looked at the `Size` class itself:

```c#
[Serializable]
[ComVisible(true)]
public struct Size
```

What? `Size` just has `Serializable` and `ComVisible`? I know they have nothing to do with the Designer - argh! But I wasn't stopping there - it was time to break out the big guns: Reflector. I drilled down to the definition of `Size`, and what I found cleared things up:

```c#
[Serializable, StructLayout(LayoutKind.Sequential), 
  TypeConverter(typeof(SizeConverter)), ComVisible(true)]
public struct Size
```

Ah, I had forgotten about `TypeConverter`. This class (as the SDK says) "provides a unified way of converting types of values to other types, as well as for accessing standard values and subproperties." I had used that before on another .NET project, but I had completely forgotten about it. Plus, I had only used it to get a list of string values to show in a drop down list - in other words, I just used a custom version of `StringConverter`, which I modified from some code on a web site I can't remember. So, I basically had to create a custom version of `TypeConverter` (call it ... say ... `DrumSetAspectsConverter`) that would give me the same effect as `SizeConverter`.

Well, rather than figuring out which methods I had to override in TypeConverte`r` (and the SDK is pretty fuzzy on this), I figured, well, I'm in Reflector - I might as well steal the implementation of `SizeConverter` and tweak it to such that it "becomes" `DrumSetAspectsConverter`. Sounded like a good plan to me! OK, first things first - I need a class definition:

```c#
public sealed class DrumSetAspectsConverter : TypeConverter
{
  // TODO...
}
```

Now I need to add a `TypeConverter` attribute on to `DrumSetAspects`:

```c#
[TypeConverter(typeof(DrumSetAspectsConverter))]
public sealed class DrumSetAspects
```

That doesn't do much to the *Properties* window. So I started to add methods from `SizeConverter` into `DrumSetAspectsConverter`. I started with `CanConvertFrom()` and `CanConvertTo()`:

```c#
using System.ComponentModel.Design.Serialization;

// ...

public override bool CanConvertFrom(
  ITypeDescriptorContext context, Type sourceType)
{
  if(sourceType == typeof(string))
  {
    return true;
  }

  return base.CanConvertFrom(context, sourceType);
}

public override bool CanConvertTo(
  ITypeDescriptorContext context, Type destinationType)
{
  if(destinationType == typeof(InstanceDescriptor))
  {
    return true;
  }
  return base.CanConvertTo(context, destinationType);
}
```

I'm not entirely thrilled with Reflector's coding style here, but I'm just going to take it as-is unless I know I need to make it specific to my design. Note that `InstanceDescriptor` is in the `System.ComponentModel.Design.Serialization` namespace. With these two methods my `Aspects` property is now editable in VS .NET:

![UserControl Property](https://jasonbock.net/images/UserControl-Property-3.png "UserControl Property")

However, I really can't do much at this point. Any change to this value results in an error, so I keep adding other overrides. The next ones are `ConvertFrom()` and `ConvertTo()`, which I had to modify a bit to match my class design:

```c#
public override object ConvertFrom(
  ITypeDescriptorContext context, 
  CultureInfo culture, object value)
{
  DrumSetAspects aspects = null;

  if(value is string)
  {
    string[] aspectsValue = 
      ((string)value).Split(
      Char.Parse(culture.TextInfo.ListSeparator));

    return new DrumSetAspects(
      bool.Parse(aspectsValue[0].Trim()),
      bool.Parse(aspectsValue[1].Trim()),
      bool.Parse(aspectsValue[2].Trim()));
  }
  else
  {
    aspects = (DrumSetAspects)base.ConvertFrom(
      context, culture, value);
  }

  return aspects;
}

public override object ConvertTo(
  ITypeDescriptorContext context, 
  CultureInfo culture, object value, 
  Type destinationType)
{
  object convertedValue = null;

  if (destinationType == null)
  {
    throw new ArgumentNullException("destinationType");
  }

  if((destinationType == typeof(string)) && 
    (value is DrumSetAspects))
  {
    DrumSetAspects aspects = (DrumSetAspects)value;

    if (culture == null)
    {
      culture = CultureInfo.CurrentCulture;
    }

    string text = culture.TextInfo.ListSeparator + " ";
    TypeConverter converter = 
      TypeDescriptor.GetConverter(typeof(bool));
    string[] textArray = new string[3];
    textArray[0] = converter.ConvertToString(
      context, culture, aspects.HasDoubleBassPedals);
    textArray[1] = converter.ConvertToString(
      context, culture, aspects.HasElectronicDrums);
    textArray[2] = converter.ConvertToString(
      context, culture, aspects.HasGong);
    convertedValue = string.Join(text, textArray);
  }
  else if((destinationType == typeof(InstanceDescriptor))
    && (value is DrumSetAspects))
  {
    DrumSetAspects aspects = (DrumSetAspects)value;
    Type[] typeArray = new Type[3] 
      {typeof(bool), typeof(bool), typeof(bool)} ;
    ConstructorInfo info = 
      typeof(DrumSetAspects).GetConstructor(typeArray);
    if(info != null)
    {
      object[] objArray = new object[3] 
        {aspects.HasDoubleBassPedals, 
        aspects.HasElectronicDrums,
        aspects.HasGong};
      convertedValue = new InstanceDescriptor(info, objArray);
    }
  }
  else
  {
    convertedValue = base.ConvertTo(
      context, culture, value, destinationType);
  }

  return convertedValue;
}
```

Essentially, what `ConvertFrom()` does is take a string like, "true, false, false" (where `ListSeparator` is the comma) and turns it into a `DrumSetAspects` instance with `HasDoubleBassPedals` equal to `true`, `HasElectronicDrums` equal to `false`, and `HasGong` equal to `false`. `ConvertTo()` works in the opposite direction (i.e. it takes a `DrumSetAspects` instance and returns a `string`), but it also returns an `InstanceDescriptor` instance that describes how a `DrumSetAspects` instance is constructed. With these two functions my VS .NET world gets closer to what I want:

![UserControl Property](https://jasonbock.net/images/UserControl-Property-4.png "UserControl Property")

I now have a string for the `Aspects` property equal to "False, False, False". If I change it to "True, False, False", `InitializeComponent()` is updated with this line of code:

```c#
private void InitializeComponent()
{
  // ...
  this.rockerDrumSet.Aspects = 
    new ComplexProperty.DrumSetAspects(true, false, false);
  // ...
}
```

Furthermore, if I type "Garbage, Garbage, Garbage" or "True - False, False", I get errors, which is good as I should only take boolean values separated by a comma. But I'm still not quite there yet. I want that tree view look so I don't have to type in a string just to change the property's value. Well, there's four more methods that I stole from `SizeDescriptor` to make this all work: `CreateInstance()`, `GetCreateInstanceSupported()`, `GetProperties()`, and `GetPropertiesSupported()`:

```c#
public override object CreateInstance(
  ITypeDescriptorContext context, 
  IDictionary propertyValues)
{
  return new DrumSetAspects(
    (bool)propertyValues["HasDoubleBassPedals"], 
    (bool)propertyValues["HasElectronicDrums"],
    (bool)propertyValues["HasGong"]);
}

public override bool GetCreateInstanceSupported(
  ITypeDescriptorContext context)
{
  return true;
}

public override PropertyDescriptorCollection GetProperties(
  ITypeDescriptorContext context, 
  object value, Attribute[] attributes)
{
  PropertyDescriptorCollection collection = 
    TypeDescriptor.GetProperties(
    typeof(DrumSetAspects), attributes);
  string[] propertyText = new string[3] 
    {"HasDoubleBassPedals", "HasElectronicDrums", 
    "HasGong"} ;
  return collection.Sort(propertyText);
}

public override bool GetPropertiesSupported(
  ITypeDescriptorContext context)
{
  return true;
}
```

The method names are self-describing. `CreateInstance()` creates a new `DrumSetAspects` instances based off of values in the `propertyValues` dictionary. I indicate that my converter supports instance creation by returning `true` from `GetCreateInstanceSupported()`. I can also return a list of property names via `GetProperties()`, and, like instance creation, I return `true` from `GetPropertiesSupported()` to indicate I return a list of property information. With this code, I finally have what I'm looking for:

![UserControl Property](https://jasonbock.net/images/UserControl-Property-5.png "UserControl Property")

I hope this little story has been helpful. It **barely** scratches the surface of the power a developer has to display information in a Designer, but I hope it gives you an overview of what can be done (and **needs** to be done!) to get things to work the way you want them to in VS .NET. Also, remember that Reflector is a **powerful** tool that can help make your .NET development life that much easier by showing you how it was done. If you want to look at this code in detail, you can get it all here - enjoy!

> Published: 01.17.2005 09:16:59 PM CST