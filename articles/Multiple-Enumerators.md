---
title: Multiple Enumerators
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Multiple Enumerators

I just ran into a situation where I had a class that needed to expose multiple enumerators. However, you can only have one `GetEnumerator()` method on your class, so ... here's how I got around that.

Here's the standard, simple example:

```c#
internal sealed class SingleDataProducer
{
  public IEnumerator<string> GetEnumerator()
  {
    yield return "one";
    yield return "two";
    yield return "three";
  }
}
```

But what if I want to have an `int` and a `string` enumeration? Well, I can solve that by having two nested classes:

```c#
internal sealed class MultipleDataProducer
{
  private List<int> intData = new List<int>(new int[] { 1, 2, 3 });
  private List<string> stringData = new List<string>(new string[] { "one", "two", "three" });

  internal IEnumerable<string> GetStringEnumerator()
  {
    return new StringEnumerator(this);
  }

  internal IEnumerable<int> GetIntEnumerator()
  {
    return new IntEnumerator(this);
  }

  private sealed class IntEnumerator : IEnumerable<int>
  {
    private MultipleDataProducer producer;

    internal IntEnumerator(MultipleDataProducer producer)
      : base()
    {
      this.producer = producer;
    }

    public IEnumerator<int> GetEnumerator()
    {
      foreach(int i in this.producer.intData)
      {
        yield return i;
      }
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
      return this.GetEnumerator();
    }
  }

  private sealed class StringEnumerator : IEnumerable<string>
  {
    private MultipleDataProducer producer;

    internal StringEnumerator(MultipleDataProducer producer)
      : base()
    {
      this.producer = producer;
    }
        
    public IEnumerator<string> GetEnumerator()
    {
      foreach(string s in this.producer.stringData)
      {
        yield return s;
      }
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
      return this.GetEnumerator();
    }
  }
}
```

By using a nested class that implements `IEnumerable<>`, I can have the containing class return an instance of the nested class and let it produce the enumeration. When I run the following console application:

```c#
class Program
{
  static void Main(string[] args)
  {
    Program.ShowSingleDataProducer();
    Console.Out.WriteLine();
    Program.ShowMultipleDataProducer();
  }

  private static void ShowMultipleDataProducer()
  {
    Console.Out.WriteLine("Multiple Producer");

    MultipleDataProducer producer = new MultipleDataProducer();
        
    foreach(string s in producer.GetStringEnumerator())
    {
      Console.Out.WriteLine(s);            
    }

    foreach(int i in producer.GetIntEnumerator())
    {
      Console.Out.WriteLine(i.ToString());
    }
  }
    
  private static void ShowSingleDataProducer()
  {
    Console.Out.WriteLine("Single Producer");
        
    SingleDataProducer producer = new SingleDataProducer();
        
    foreach(string s in producer)
    {
      Console.Out.WriteLine(s);
    }
  }
}
```

I get these results:

```
Single Producer
one
two
three

Multiple Producer
one
two
three
1
2
3
```

Now, I'm not saying this result is pretty. Something about it seems too complex to me. But ... it gets the job done. I'm open to solutions that are more elegant!

> Published: 10.09.2007 08:21:37 AM CST