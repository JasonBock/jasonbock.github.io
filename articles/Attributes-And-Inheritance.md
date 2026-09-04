---
title: Attributes and Inheritance
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Attributes and Inheritance

One aspect of attributes that always bugged me (and still does - why else would I be devoting a blog entry to it?) is how attribute discovery works in inheritance scenarios. Here's an example that uses class-based attributes. Let's say you create two attributes:

```c#
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false, 
  Inherited = true)]
public sealed class InheritableClassAttribute : Attribute {}

[AttributeUsage(AttributeTargets.Class, AllowMultiple = false, 
  Inherited = false)]
public sealed class NonInheritableClassAttribute : Attribute {}
```

The key difference between the two attributes is the value of `Inherited`. The first class sets it to `true` and the second class sets it to `false` (note that if you don't explicitly set it, it will be `true`). As you can gather by the property name, this value affects how attributes are found when the base class is used in inheritance hierarchies, but (in my mind at least) the .NET SDK doesn't do a great job in illustrating the ramifications of changing this property value. Let's create a class that use these attributes and then use it as a base class for two other classes - one that redefines the attributes on the class, and one that does not:

```c#
[InheritableClass(), NonInheritableClass()]
public class BaseClassWithClassAttributes {}

[InheritableClass(), NonInheritableClass()]
public class SubClassWithClassAttributes : BaseClassWithClassAttributes {}

public class SubClassWithoutClassAttributes : BaseClassWithClassAttributes {}
```

Now, when you read the SDK (specifically, this section), it will (effectively) tell you that `SubClassWithoutClassAttributes` will have `InheritableClass` but not `NonInheritableClass`. But what I've found is that it's dependent on how you look at the derived class via the Reflection API. There's a method called `IsDefined()` that takes a boolean value as a second parameter that tells the code to look up the inheritance hierarchy for the existence of the attribute to determine if it's defined on the member that you're currently looking at. Here's a unit test to illustrate how this works:

```c#
[TestFixture()]
public sealed class SubClassWithoutClassAttributesTests
{
  private Type inheritableClassType = 
    typeof(InheritableClassAttribute);
  private Type nonInheritableClassType = 
    typeof(NonInheritableClassAttribute);
  private Type subClassWithoutClassAttributesType = 
    typeof(SubClassWithoutClassAttributes);

  [Test()]
  public void GetSubClassAttributesWithInherit()
  {
    Assert.IsTrue(subClassWithoutClassAttributesType.IsDefined(
      this.inheritableClassType, true));
    Assert.IsFalse(subClassWithoutClassAttributesType.IsDefined(
      this.nonInheritableClassType, true));
  }

  [Test()]
  public void GetSubClassAttributesWithoutInherit()
  {
    Assert.IsFalse(subClassWithoutClassAttributesType.IsDefined(
      this.inheritableClassType, false));
    Assert.IsFalse(subClassWithoutClassAttributesType.IsDefined(
      this.nonInheritableClassType, false));
  }
}
```

Study that code hard, and you'll start to see (at least I hope you do!) what my issue is. The first test tells `IsDefined()` to look at the current type first to see if the attribute exists. In this case, `SubClassWithoutClassAttribute` does not have the attributes in question, but `BaseClassWithClassAttributes` does. Remember, though, that `NonInheritableClassAttribute` was designed to not be included in inheritance hierarchy searches, so it will not show up as an attribute on `SubClassWithoutClassAttribute`. The second test shows that if you do not look up the inheritance hierarchy, neither attribute will show up on `SubClassWithoutClassAttribute` **even though** `InheritableClassAttribute` is designed to be included in inheritance hierarchy searches.

This is a key point to keep in mind. If you design an attribute to only be associated with the current member and not any subclasses, then it doesn't matter how someone tries to find it via `IsDefined()`. A good example of this is `SerializableAttribute` - it does not carry over to subclasses, and this is a good thing as it forces the developer to be explicit in selecting which classes can support serialization and which cannot. A base class may support serialization, but a subclass may have sensitive information in it and the developer does not want that information showing up in a serialization stream. The developer doesn't have to do anything in the subclass though; in fact, s/he would have to mark the subclass again with `SerializableAttribute` so the subclass could support serialization. But, if you allow your attribute to be included in hierarchy searches, it still may not be found if inherit is set to `false`. It feels like there's tension between what the designer intented and what a client sees with a search when an attribute allows itself to be included in hierarchical searches. `IsDefined()` seems to be set up for some optimizations, as an inheritance search will take longer to find that attribute than if you just look at the class in question. One way to ensure searches will always find an inheritable attribute is to always define that attribute on any class that should have it. But that seems to defeat the purpose of having attribute that can be inheritable!

I don't think I'm missing anything in my code, but I'm confused as to when one should and should not do a full heirarchy search. Any ideas, please let me know (and yes, comments are coming soon, very soon ... ).

> Published: 12.30.2004 04:02:28 PM CST