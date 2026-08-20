---
title: Delegates, Nested Classes and Refactoring ... What Could Go Wrong?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Delegates, Nested Classes and Refactoring ... What Could Go Wrong?

A lot.

Take a look at the following C# code:

```c#
public class Base
{
}

public class LeftMiddle : Base
{
  public static int SortEmbeddeds(Embedded x, Embedded y)
  {
    return 0;
  }
    
  public class Embedded
  {
  }
}

public class RightMiddle : Base
{
  public static int SortEmbeddeds(Embedded x, Embedded y)
  {
    return 0;
  }

  public class Embedded
  {
  }
}

public class LeftBottom : LeftMiddle
{
  public void DoSomething()
  {
    List<Embedded> embeds = new List<Embedded>();
    embeds.Sort(LeftMiddle.SortEmbeddeds);
  }
}

public class RightBottom : RightMiddle
{
  public void DoSomething()
  {
    List<Embedded> embeds = new List<Embedded>();
    embeds.Sort(RightMiddle.SortEmbeddeds);
  }
} 
```

OK, the names aren't very nice, but the thing to note is that the 2 classes in the middle of the inheritance hierarchy have duplicated code via the nested `Embedded` class and the `SortEmbeddeds()` method. So, it makes sense to move that code up into `Base`, right?

```c#
public class Base
{
  public static int SortEmbeddeds(Embedded x, Embedded y)
  {
    return 0;
  }

  public class Embedded
  {
  }
}
```

And, of course, change the bottom classes accordingly:

```c#
public class LeftBottom : LeftMiddle
{
  public void DoSomething()
  {
    List<Embedded> embeds = new List<Embedded>();
    embeds.Sort(Base.SortEmbeddeds);
  }
}

public class RightBottom : RightMiddle
{
  public void DoSomething()
  {
    List<Embedded> embeds = new List<Embedded>();
    embeds.Sort(Base.SortEmbeddeds);
  }
}
```

Well, when I did that, I got a really bizarre error:

```
Error 3 Argument '1': cannot convert from 'method group' to 'System.Collections.Generic.IComparer<MethodOfDelegateError.RightMiddle.Embedded>' C:\JasonBock\Personal\.NET Projects\MethodOfDelegateError\Classes.cs 43 16 MethodOfDelegateError
```

Actually, I didn't show you the complete refactoring, and if I highlight the critical information in the error message you'll figure out what I did wrong:

```
Error 3 Argument '1': cannot convert from 'method group' to 'System.Collections.Generic.IComparer<MethodOfDelegateError.RightMiddle.Embedded>' C:\JasonBock\Personal\.NET Projects\MethodOfDelegateError\Classes.cs 43 16 MethodOfDelegateError
```

The problem was I moved the code out of `LeftMiddle` so it looked like this:

```c#
public class LeftMiddle : Base
{
}
```

But I only removed the method from `RightMiddle`; it still has a nested `Embedded` class:

```c#
public class RightMiddle : Base
{
  public class Embedded
  {
  }
}
```

Once I removed the nested type as well, everything compiled.

I just ran into this error today, and if I would've just read the damn error message, things would've fixed a lot quicker. I've had this "method group" error before in projects and it always seems to hint that there's something kind of odd going on in your code that this error is kind of "masking" (for lack of a better term). The code base is not trivial and I just flat-out missed the fact that I forgot to move the other nested class, and I was just glossing over the error message. Kind of a waste of time, although fortunately I've learned that rather than beating my head against the desk for hours and hours it's better to ask someone to take a look at the problem. I did that today and we found the issue in less than 2 minutes.

> Published: 08.23.2007 11:24:43 AM CST