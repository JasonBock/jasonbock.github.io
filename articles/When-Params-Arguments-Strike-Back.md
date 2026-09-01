---
title: When Params Arguments Strike Back
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When Params Arguments Strike Back

Let's say, once upon a time (just run with me for a while on this) a developer creates a class called `StupidClass` in an assembly called ... oh, let's say it was called `ParamsSnafuServer`:

```c#
public class StupidClass
{
  public void StupidMethod(params string[] stupidArguments) {}
}
```

Now a client ( ... `ParamsSnafuClient` ... ) has a form that calls it:

```c#
public MainForm()
{
  this.InitializeComponent();
  StupidClass stupidClass = new StupidClass();
  stupidClass.StupidMethod("a", "b", "c", "d");
}
```

All is well with the world in version 1.0, and the bunnies are sleeping close to the foxes, content in the fact that the code compiled. Now, in 2.0, the super-genius developer who maintains `ParamsSnafuServer` (I won't say who did this because I would incriminate myself) overloads `StupidMethod()`:

```c#
public class StupidClass
{
  public void StupidMethod(params string[] stupidArguments) {}

  public void StupidMethod(int stupidInt, 
    params string[] stupidArguments) {}
}
```

Both the server and client compile for 2.0, and now the bunnies are giggling and snickering at the foxes - they're getting a little cocky. In version 3.0, the dumbass developer (again, I won't mention my name) decides to get adventerous:

```c#
public class StupidClass
{
  public void StupidMethod(params string[] stupidArguments) {}

  public void StupidMethod(int stupidInt, 
    params string[] stupidArguments) {}

  public void StupidMethod(string stupidString, int stupidInt, 
    params string[] stupidArguments) {}
}
```

Whew! A recompile for the server and client for version 3.0, and the bunnies are now taunting the foxes with their silly dances. Unfortunately, the big dumbass developer (you still don't know it was me, right?!) goes completely awry in version 4.0:

```c#
public class StupidClass
{
  public void StupidMethod(params string[] stupidArguments) {}

  public void StupidMethod(int stupidInt, 
    params string[] stupidArguments) {}

  public void StupidMethod(string stupidString, int stupidInt, 
    params string[] stupidArguments) {}

  public void StupidMethod(string stupidOverload, 
    params string[] stupidArguments) {}
}
```

Argh! Bunny blood and fur goes flying everywhere!!!! `ParamsSnafuClient` hates me now!!!!!

```
C:\jbock\Personal\.NET Projects\ParamsSnafu\ParamsSnafuClient\StupidForm.cs(19): The call is ambiguous between the following methods or properties: 'ParamsSnafuServer.StupidClass.StupidMethod(string, params string[])' and 'ParamsSnafuServer.StupidClass.StupidMethod(params string[])'
```

And the dumbass developer is left cleaning up bunny fluids and bone fragments.

Moral of the story? Be very careful if you overload a method that has a `params` value. In particular, make sure that the preceding argument is not of the same type as the one used for the `params` argument, because then the compiler can't figure out which overload you're trying to call.

Damn, I hate cleaning up bunny blood ...

> Published: 02.02.2005 02:22:32 PM CST