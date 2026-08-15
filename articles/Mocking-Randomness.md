---
title: Mocking Randomness
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mocking Randomness

One of things I've run into a lot lately during my off-hours coding and study endeavours is the need to use randonmess. With evolutionary programming you really need randomness to shake things up while things are executing. However, trying to test code that's reliant on randonmess is just impossible ... if you don't have dependency injection built in. However, if you have it there, things become far easier to test.

Let's look a simple piece of code that gives you a random direction:

```c#
public enum Direction
{
  North,
  South,
  East,
  West
}

public static class DirectionSelector
{
  private static readonly int numberOfDirections = 
    Enum.GetValues(typeof(Direction)).Length;
    
  public static Direction Select()
  {
    return (Direction)(new Random()).Next(DirectionSelector.numberOfDirections);
  }
}
```

> Note: I'm using the `Random` class here because it's simple to use, but it's definitely not an "industrial-strength" random number generator. If you need that (e.g. for crytographic purposes), use the `RandomNumberGenerator` class in `System.Security.Cryptography`.

Now, how would I write a test against this code to ensure I get a north direction when the random number generator returns zero? I can write the test:

```c#
[TestMethod]
public void GetNorthDirection()
{
  Assert.AreEqual(Direction.North, DirectionSelector.Select());
}
```

but it's completely worthless, because it's going to fail on average 75% of the time.

However, if we change `Select()` to take a `Random` reference and tweak things a bit:

```c#
public static class DirectionSelector
{
  private static readonly int numberOfDirections = 
    Enum.GetValues(typeof(Direction)).Length;
    
  public static Direction Select()
  {
    return DirectionSelector.Select(new Random());
  }

  public static Direction Select(Random random)
  {
    return (Direction)random.Next(DirectionSelector.numberOfDirections);
  }
}
```

Now I can create a subclass of `Random` where I have complete control over what `Next()` will return:

```c#
[TestClass]
public sealed class DecisionMakerTests
{
  [TestMethod]
  public void GetEastDirectionWithMock()
  {
    Assert.AreEqual(Direction.East,
      DirectionSelector.Select(new MockRandom((int)Direction.East)));
  }

  [TestMethod]
  public void GetNorthDirectionWithMock()
  {
    Assert.AreEqual(Direction.North, 
      DirectionSelector.Select(new MockRandom((int)Direction.North)));
  }

  [TestMethod]
  public void GetSouthDirectionWithMock()
  {
    Assert.AreEqual(Direction.South,
      DirectionSelector.Select(new MockRandom((int)Direction.South)));
  }

  [TestMethod]
  public void GetWestDirectionWithMock()
  {
    Assert.AreEqual(Direction.West,
      DirectionSelector.Select(new MockRandom((int)Direction.West)));
  }

  private sealed class MockRandom : Random
  {
    private int value;
        
    public MockRandom(int value)
      : base()
    {
      this.value = value;
    }

    public override int Next(int maxValue)
    {
      return this.value;
    }
  }
}
```

Cool! Now all the tests pass.

Of course, you can use a mocking framework to achieve the same effect. But either way, having the control over the resource makes it far easier to ensure the code works as expected.

> Published: 04.10.2009 01:24:24 PM CST