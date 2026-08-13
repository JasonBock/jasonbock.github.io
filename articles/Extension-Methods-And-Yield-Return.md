---
title: Extension Methods and "yield return"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Extension Methods and "yield return"

I've created numerous extensions in the past like this:

```c#
public static void EnumerateNodes(this Expression @this)
{
  if(@this == null)
  {
    throw new ArgumentNullException("this");
  }

  for(var i = 0; i < @this.GetNodeCount(); i++)
  {
    // Do something fun with each node...
  }
}
```

What I do in the method isn't that big of a deal - it's that I can ensure that I don't get a non-null argument. I've been able to write tests to confirm that a null argument causes the expected exception to be thrown:

```c#
[TestMethod, ExpectedException(typeof(ArgumentNullException))]
public void EnumerateNodesWithNullArgument()
{
  (null as Expression).EnumerateNodes();
}
```

But today ... well, something changed. I created an extension method like this:

```c#
public static IEnumerable<Expression> EnumerateNodes(this Expression @this)
{
  if(@this == null)
  {
    throw new ArgumentNullException("this");
  }

  for(var i = 0; i < @this.GetNodeCount(); i++)
  {
    yield return @this.GetNode(i);
  }
}
```

The bizarre thing is my test would fail because I'd never get the exception.

What gives?

Opening up the assembly in Reflector showed some very unexpected results:

```c#
public static IEnumerable<Expression> EnumerateNodes(this Expression @this)
{
  return new <EnumerateNodes>d__0(-2) { <>3__this = @this };
}
```

Wow. My null check is gone.

Now I know when you use "yield return" all sorts of freaky things happen underneath the scenes to ensure things work correctly. Basically, what this code is doing is just capturing the argument and only doing something with it when an enumeration occurs.

**That's** the problem.

I need to actually **do** something with the return value. So if I change my test code to this:

```c#
[TestMethod, ExpectedException(typeof(ArgumentNullException))]
public void EnumerateNodesWithNullArgument()
{
  foreach(var node in (null as Expression).EnumerateNodes()) { }
}
```

Now the test succeeds as expected.

Actually, if you dive into the implementation of `MoveNext()` for `<EnumerateNodes>d__0`, you'll find out where that null check went:

```c#
private bool MoveNext()
{
    switch (this.<>1__state)
    {
        case 0:
            this.<>1__state = -1;
            if (this.@this == null)
            {
                throw new ArgumentNullException("this");
            }
```

So it didn't get compiled away ... just moved around a bit.

> Published: 06.22.2010 01:37:33 PM CST