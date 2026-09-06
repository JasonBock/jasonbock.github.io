---
title: Three Class Issues in VS .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Three Class Issues in VS .NET

It's late at night, but today I've found the third weird issue in VS .NET 2003 with classes over the last year or so. I'm sure others have run into these, although the third one is a really odd case, but I wanted to publish them anyway so you can be aware of these problems.

The first one consists of abstract user controls:

```c#
namespace Viktor.ClassFirstTests
{
  public abstract class AbstractControl : UserControl {}
}
```

If you open this class in the Designer, you'll have no problems. However, you will run into problems once you make a concrete class use `AbstractControl` as its base class:

```c#
namespace Viktor.ClassFirstTests
{
  public abstract class ImplementingControl : AbstractControl {}
}
```

If you try to open `ImplementingControl` in the Designer, you'll get an error message:

```
The designer must create an instance of type 'Viktor.ClassFirstTests.AbstractControl' but it cannot because the type is declared as abstract.
```

There's an easy way around this: make a shim class that sits between `AbstractControl` and `ImplementingControl`:

```c#
namespace Viktor.ClassFirstTests
{
  public abstract class ShimControl : AbstractControl {}

  public abstract class ImplementingControl : ShimControl {}
}
```

This isn't an ideal solution, because any abstract methods and properties in `AbstractControl` will need implemetations in `ShimControl` (most likely no-op implementations), and then `ImplementingControl` needs to override these no-op implementations.

Moral of the first story? **Be careful if you make abstract controls.**

Here's the second issue:

```c#
namespace Viktor.ClassFirstTests
{
  public class Interference {}

  public class NotTheFirstClass : System.Windows.Forms.UserControl {}
}
```

If you try to open `NotTheFirstClass` in the Designer, you'll get the following error message:

```
The class NotTheFirstClass can be designed, but is not the first class in the file. Visual Studio requires that designers use the first class in the file. Move the class code so that it is the first class in the file and try loading the designer again.
```

Again, a simple fix: put the `Interference` class after the definition of `NotTheFirstClass`, or (better yet in my book), in another code file.

Moral of the second story? **Don't put class definitions ahead of control definitions in a file.**

Now the last one is really subtle:

```c#
namespace ClassFirstTests
{
  public delegate void InterferingDelegate();

  public class ResourceInterference : System.Windows.Forms.UserControl {}
}
```

Note that the namespace has been shortened from `Viktor.ClassFirstTests` to `ClassFirstTests`. Also, the *Default Namespace* project property is `Viktor.ClassFirstTests`. So what's the problem here? Simple - set `Localizable` on `ResourceInterference` to `true`, compile, and try to put `ResourceInterference` on a form. Uh, oh...:

![Three Class Issues](https://jasonbock.net/images/Three-Class-Issues-1.png "Three Class Issues")

I had to trim the message box a bit, but you can see that there's something wrong with the resource naming in the assembly. First, check out the generated code in `InitializeComponent()` for `ResourceInterference`:

```c#
private void InitializeComponent()
{
  System.Resources.ResourceManager resources = 
    new System.Resources.ResourceManager(typeof(ResourceInterference));
  this.AccessibleDescription = 
    resources.GetString("$this.AccessibleDescription");
  // ...
}
```

Since the control is localizable, the generated code initializes the control based on resource values. Note that the `ResourceManager` instance is based off of a type, which in this case has a full name of `ClassFirstTests.ResourceInterference`. **However**, check out what the resource is really named (via a trip to Reflector):

![Three Class Issues](https://jasonbock.net/images/Three-Class-Issues-2.png "Three Class Issues")

For some reason, VS .NET will use the *Default Namespace* value if the class is not the first class in the code file. I know, it sounds weird, but if you remove `InterferingDelegate` from the picture and recompile, everything will work as expected.

Moral of the third story? It's just like the second, but even more limiting: **Strongly consider only having one class definition per file.**

> Published: 12.16.2004 01:49:35 AM CST