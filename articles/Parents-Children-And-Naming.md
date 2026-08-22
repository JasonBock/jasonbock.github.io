---
title: Parents, Children, and Naming
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Parents, Children, and Naming

I just ran into a very odd C# coding situation. Here's the problem. Let's say I have an `IChild` interface:

```c#
using System;

namespace ParentsChildrenAndNaming
{
  public interface IChild { }
}
```

Now I define a `Child` class with a factory method:

```c#
using System;

namespace ParentsChildrenAndNaming
{
  public sealed class Child : IChild
  {
    private Child() : base() { }
        
    public static Child GetChild()
    {
      return new Child();
    }
  }
}
```

Now I define a `Parent` class with a `Child` property and it also has a factory method:

```c#
using System;

namespace ParentsChildrenAndNaming
{
  public sealed class Parent
  {
    private IChild child;
        
    private Parent() : base() { }
        
    public static void Create()
    {
      Parent parent = new Parent();
      <b>parent.child = Child.GetChild();</b>
    }
        
    public IChild Child
    {
      get
      {
        return this.child;
      }
      set
      {
        this.child = value;
      }
    }
  }
}
```

The code in bold is not compiler-friendly:

```
Error 1 An object reference is required for the nonstatic field, method, or property 'ParentsChildrenAndNaming.Parent.Child.get'
Error 2 'ParentsChildrenAndNaming.IChild' does not contain a definition for 'GetChild'
```

Now, when I first saw this, I was completely befuddled. `Child` has a static `GetChild` method, so ... what's the problem? Here's the kicker. `Parent` contains a `Child` property (note the similarity of names here). Even though the property is not static, the compiler thinks that the "Child" in the static `Create` method in `Parent` is the property, not the `Child` class. Argh!

This feels wrong. First, you don't have access to the `Child` property in that static method anyway, so why would the compiler be so confused? The only way I could get around this is to fully-qualify the class name:

```c#
parent.child = ParentsChildrenAndNaming.Child.GetChild();
```

... or I could change the name of the class. The case where this came up is with `Principal` and `Identity` in `System.Security.Principal`. The `IPrincipal` interface has an `Identity` property, and I created a class called ... what else? `Identity`. Of course, now I have to rename the class to ... what? `MyIdentity`? `CustomIdentity`? I don't like either of those, just like I don't like naming base class with the "Base" suffix. So ... I guess I'll go with `PrettyIdentity`.

Yeah. I like that.

> Published: 03.14.2007 10:30:38 AM CST