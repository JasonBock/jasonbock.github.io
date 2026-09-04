---
title: Constraining Interface Implementations to a Specific Type
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Constraining Interface Implementations to a Specific Type

Lately I've been getting a bit frustrated because I need to extend a base class that I can't change (`Control`). But this post is not to (directly) complain about my pain; rather, this post is all about proposing a new language feature in C# (and potentially .NET in general) [1]. This isn't a formal description; it's pretty much an off-the-cuff idea I had yesterday, but, well, I thought it would be fun to show it to the world and let it flail in the winds.

Let's say you have the following interface:

```c#
interface IReshapedControl
{
  void Reshape();
}
```

The intent of this interface is that it would be used on a number of classes that all have an (indirect) common base class [2], `Control`:

```c#
class CustomPanel : Panel, IReshapedControl {}
class CustomButton : Button, IReshapedControl {}
class CustomUri : Uri, IReshapedControl {}
```

As the 3rd line of code demonstrates, I have no way to enforce that the interface is only applied to `Control`-based objects. This becomes problematic when I have to write a helper function that has an argument of type `IReshapedControl`:

```c#
void GeneralReshaping(IReshapedControl control)
{
  control.Reshape();

  Control underlyingControl = control as Control;

  if(underlyingControl != null)
  {
    foreach(Control childControl in underlyingControl.Controls)
    {
      if(childControl is IReshapedControl)
      {
        GeneralReshaping((IReshapedControl)childControl);
      }
    }
  }
}
```

This helper function will reshape a control and its child controls, provided the child control implements `IReshapedControl`. But, as you can see, I have to do some casts back and forth to do all of this work. What would be nice is if I could constrain the interface such that it could only be implemented on classes of a specific type:

```c#
interface IReshapedControl constrained Control
{
  void Reshape();
}
```

What the constrained keyword does is specify that the interface can only be implemented on classes that are either of the type given or a subtype. Therefore, we could no longer use `IReshapedControl` on `CustomUri`:

```c#
class CustomPanel : Panel, IReshapedControl {}
class CustomButton : Button, IReshapedControl {}

//  This line of code wouldn't compile.
//  class CustomUri : Uri, IReshapedControl {}
```

By constraining the interface, my `GeneralReshaping` method becomes a bit smaller:

```c#
void GeneralReshaping(IReshapedControl control)
{
  control.Reshape();

  foreach(Control childControl in control.Controls)
  {
    if(childControl is IReshapedControl)
    {
      GeneralReshaping((IReshapedControl)childControl);
    }
  }
}
```

Since `IReshapedControl` can only be applied to `Control` types, the foreach statement is valid as there will be a `Controls` property on the control reference.

This idea is useful when you don't have the ability to change a base class (i.e. it's part of a compiled assembly) and you want subclasses of that class to have certain characteristics. Defining an interface enforces what these new subclasses will do, but it leads to code duplication. Moving code into helper methods (like `GeneralReshaping()`) helps, but what I've found is these helper methods want the argument to "act" like the interface and the base class. Therefore, you have to use casts and it's possible that the method may be invoked with an object reference of the "wrong" type. [3]

Another notational option is use an attribute:

```c#
[Constrained(typeof(Control))]
interface IReshapedControl
{
  void Reshape();
}
```

I like the keyword approach better, but that would require a C# language change, whereas an attribute wouldn't do that (although Intellisense and the C# compiler would have to get "smarter" to understand what a constrained interface implies, but that would be the case whether you use a keyword or an attribute).

As a side note, I thought about having multi-constrained interfaces, but that doesn't seem to buy anything. Take a look at this:

```c#
interface IReshapedControl constrained Control, Uri
{
  void Reshape();
}

class CustomPanel : Panel, IReshapedControl {}
class CustomButton : Button, IReshapedControl {}
class CustomUri : Uri, IReshapedControl {}

void GeneralReshaping(IReshapedControl control)
{
  control.Reshape();

  foreach(Control childControl in control.Controls)
  {
    // What if control is of type Uri??
  }
}
```

Now I'm in a situation where I don't know for sure what the type of control is, so I can't assume there will be a `Controls` property on the given argument. Basically, I'm back to where I am now!

To be honest, I'm not entirely comfortable with this idea. Having an interface know something about which classes it can be implemented on seems bass-ackwards. Even if this never makes it as a C# addition (and I'm not holding my breath), one could write an FxCop rule that would ensure interfaces marked with ConstrainedAttribute metadata were only implemented on correct classes, but that only goes so far, especially if the interface is public. How would you be able to ensure every implementing class of that interface was a correct one? Oh, well, it was an interesting idea to me.

[1] Believe me, I'm not that gullable to think this will ever be considered as a new language feature, but one can dream dreamy dreams ...

[2] My post originally had the classes descending directly from Control, which was incorrect - I've fixed all of the code examples, which should clarify the problem.

[3] Although the cost of a cast may be far cheaper than it would be to handle constrained interfaces.

> Published: 01.11.2005 06:13:02 PM CST