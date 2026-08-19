---
title: Constructors, Virtual Members, and Initialization
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Constructors, Virtual Members, and Initialization

Recently at the client, someone asked me the following question:

"How do you get around having a constructor of an abstract class call an abstract method?"

I had to smile a bit, because this problem is one that I've noticed popping up now and then. Here's a code snippet to illustrate this:

```c#
public class InvalidStateException : Exception
{
  // Assume the custom exception is implemented properly...
}

public class Base
{
  public Base()
    : base()
  {
    this.RunCheck();
  }

  protected virtual void RunCheck() 
  {
  }
}

public class Derived : Base
{
  public Derived()
    : base()
  {
    this.Value = 418;
  }

  protected override void RunCheck()
  {
    if(this.Value == 0)
    {
      throw new InvalidStateException("Object is in the wrong state.");
    }
  }
    
  public int Value
  {
    get;
    private set;
  }
}
```

(NOTE: I've pulled this code from Brad's site, with some minor modifications.)

When the following code is run:

```c#
Derived d = new Derived();
```

we get an `InvalidStateException`. Calling virtual members in a constructor is verboten unless you are absolutely sure you know it's "safe" to invoke that member (see the Code Analysis docs for details). That's not an easy guarantee to satisfy. At the same time, though, it would be nice if developers had a way to have a "constructor" where it was OK to invoke virtual members. Here's a number of ideas and the consequences of choosing those ideas. Some of them may be very far-fetched (e.g. some of them would require changes to the BCL and/or CLR) but I wanted to run though all the options that I could come up with.

What we really want is a virtual method (let's call it `Initialize()`) that a developer can override to use virtual members in a safe fashion. The guarantee that must be fulfilled is that is has to be called immediately after the constructor has finished (we'll get to that part later on). One idea is to add this method to the object class:

```c#
public class Object
{
  protected virtual void Initialize() {}
}
```

While there are some aspects of this approach that I like, overall I think it's a very bad idea. The key reason is that the problem of virtual member usage in constructors is not a common problem, and adding a virtual method to the object class seems unnecessary. It's probably why `Dispose()` wasn't added to `object` directly (among other reasons, of course), because more often than not you don't need to address the issues of disposing unmanaged resouces, so why add a method to the core class of .NET when most of the time you don't need it?

So let's define an interface, `IInitializable`, that has one method, `Initialize()`:

```c#
public interface IInitializable
{
  void Initialize() {}
}
```

That was the simple part. Granted, this requires an addition to the BCL to do this, but for now let's assume that this was done. Let's update our classes to implement this interface:

```c#
public class ObjectInitializedException : Exception
{
  // Assume the custom exception is implemented properly...
}

public class Base : IInitializable
{
  private bool isInitialized;

  public Base()
    : base()
  {
  }

  public void Initialize()
  {
    if(this.isInitialized)
    {
      throw new ObjectInitializedException();
    }

    this.RunCheck();
    this.isInitialized = true;
  }

  protected virtual void RunCheck() 
  {
  }
}

public class Derived : Base
{
  public Derived()
    : base()
  {
    this.Value = 418;
  }

  protected override void RunCheck()
  {
    if(this.Value == 0)
    {
      throw new InvalidStateException("Object is in the wrong state.");
    }
  }

  public int Value
  {
    get;
    private set;
  }
}
```

The isInitialized flag ensures that the object is initalized only once.

Here comes the meat of the issue: satifying the guarantee of invoking `Initalize()` after the constructor finished. Right now, we haven't done anything to improve the situation (actually, it's worse because `RunCheck()` is no longer called!); we need something to call `Initalize()`. The simple thing is to rely on the developer creating the object:

```c#
Derived d = new Derived();
d.Initialize();
int value = d.Value;
```

Of course, this will never do: developers are forgetful creatures that don't always do the things they're supposed to do. That's why C# has keywords like "lock", "foreach", and "using" because the compiler generates code for us that does the right thing. So what can we do to ensure `Initialize()` is called correctly?

One option is introducing a keyword, like "initialize":

```c#
initialze(Derived d = new Derived())
{
  int value = d.Value;
}
```

However, since "using" is already overloaded (you can use it to specify a namespace you want to use and to ensure a disposable object is disposed), let's overload it even more:

```c#
using(Derived d = new Derived())
{
  int value = d.Value;
}
```

What this would do is generate code during compilation that would equate to something like this:

```c#
Derived d = new Derived();
d.Initialize();
int value = d.Value;
```

The nice thing about this idea is it would create the code for you. It would also work for initializating and disposing an object if it implemented `IInitializable` and `IDisposable` (which is why I didn't add an exception handler in the last code snippet). The problem with this is that it's C#-specific. It doesn't matter if we used "using" or another keyword; the point is something in the compiler "understands" what it should do when it sees the "using" token and generates the appropriate code. It basically becomes a requirement for every language to do something like this. (Think about supporting "using" in a language that doesn't have it - you end up having to write a bunch of code that just becomes boilerplate and you'd end up wondering "why doesn't my language support this??")

Another is to write a Code Analysis rule that would look for objects that implement `IInitializable` and yell at the developer when they don't call `Initialize()` right away:

```c#
// Wrong! Initialize() needs to be called.
Derived d = new Derived();
int value = d.Value;
```

or that the don't do it in the right order:

```c#
// Wrong! Initialize() needs to be called right after the constructor.
Derived d = new Derived();
int value = d.Value;
d.Initialize();
```

The problem with this is that very few developers don't use Code Analysis (and they should, but that's an entirely different matter). Even if every .NET developer used Code Analysis, it's too easy to ignore, and with this design, you really don't want to ignore calling this method.

Yet another option would be to impose some kind of compiler "action", whether this was done directly in the compiler. The problem with this is that it would have to become a core feature of every .NET compiler.

Another is to use a post-build code injection process that uses Cecil or some other assembly modification library. The problem with this is that there is no standard way to do this.

Of course, you could use static, creational methods to do all the work in the right order, but that's something that the developer has to put together correctly. Having the `IImplementable` idea seems to compartmentalize things nicely.

What I'm driving at is that, for this to work properly, it needs to "just work". That is, I think it would have to be an over-arching .NET feature to be successful. The client code would still look like this:

```c#
Derived d = new Derived();
int value = d.Value;
```

but the CLR would be "IInitializable-aware" and change the IL of the method at JIT time from this:

```
.locals init (
 [0] class Initialization.Derived d)
L_0000: newobj instance void Initialization.Derived::.ctor()
L_0005: stloc.0 
L_0006: ldloc.0 
L_0007: callvirt instance int32 Initialization.Derived::get_Value()
L_000c: pop 
L_000d: ret 
```

to this:

```
.locals init (
 [0] class Initialization.Derived d)
L_0000: newobj instance void Initialization.Derived::.ctor()
L_0005: stloc.0 
L_0006: ldloc.0 
L_0007: callvirt instance void Initialization.Base::Initialize()
L_000c: ldloc.0 
L_000d: callvirt instance int32 Initialization.Derived::get_Value()
L_0012: pop 
L_0013: ret 
```

before it is executed. That is, it would see when the `newobj` opcode is used, and check to see if the object's class implements `IInitializable`. If it does, call `Initialize()` right after `newobj` is done.

Of course, one could say, "well then, why isn't the same thing done for disposable objects?" Remember that it's not the job of the CLR to determine when an object should be disposed. However, in this case I think it's reasonable for the CLR to know when `Initialize()` should be called.

I have to admit, this is kind of a rambling post. But I've seen this issue of virtual member usage in constructors enough pop up enough in code bases that I've worked on that I'd like to see a "standard" approach to handling it. There's always a way to unwind the knot and have a clean solution; having an interface like `IInitializable` that would work for object initialization would be clean and easy to understand. The only problem is ensuring that `Initialize()` is called correctly.

(Or we could just change what `newobj` does - see this article for a fairly unrelated yet interesting article on new).

> Published: 01.02.2008 01:49:40 PM CST