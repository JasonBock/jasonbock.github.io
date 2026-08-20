---
title: Circular Code Analysis
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Circular Code Analysis

I write this test: 

```c#
[TestMethod, ExpectedException(typeof(LineException))]
public void CreateLineCornerToEdge()
{
  new Line(new Coordinate(0, 0, 0), new Coordinate(0, 2, 4));
}
```

Then Code Analysis barks at me:

```
Error 1 CA1806 : Microsoft.Usage : LineTests.CreateLineCornerToEdge():Void creates an instance of Quixo3D.Engine.Negascout.Line which is either not assigned to a variable or is never used. Remove the object creation if it is unnecessary or use it within the method. C:\JasonBock\Personal\.NET Projects\Quixo3D\Quixo3D.Engine.Tests\Negascout\LineTests.cs 39 Quixo3D.Engine.Tests
```

So I change the test:

```c#
[TestMethod, ExpectedException(typeof(LineException))]
public void CreateLineCornerToEdge()
{
  Line line = new Line(new Coordinate(0, 0, 0), new Coordinate(0, 2, 4));
}
```

Then I get this error:

```
Error 1 CA1804 : Microsoft.Performance : LineTests.CreateLineCornerToEdge():Void declares a local, 'line', of type Quixo3D.Engine.Negascout.Line, which is never used or is only assigned to. Use this local or remove it. C:\JasonBock\Personal\.NET Projects\Quixo3D\Quixo3D.Engine.Tests\Negascout\LineTests.cs 38 Quixo3D.Engine.Tests
```

Sigh ... "Suppress Message" time!

> Published: 10.17.2007 08:14:42 PM CST