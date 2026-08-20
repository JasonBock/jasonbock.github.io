---
title: Simple Code Refactoring
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Simple Code Refactoring

As I wait for our development servers to come back online, I thought I'd share a very simple code refactoring I did last night. I have no idea if you'd call this IoC or what (I'm not big on patterns and architectury talk) but it made things a lot easier.

I wanted to do some simple timing tests in a console app for an upcoming talk. Here was the original code:

```c#
private static void TimeConversionIntEnumerationDynamicMethod()
{
  Stopwatch stopwatch = Stopwatch.StartNew();
  DynamicMethodEnumConverter converter = new DynamicMethodEnumConverter();
    
  for(int i = 0; i < 500000; i++)
  {
    foreach(object value in Enum.GetValues(typeof(IntEnumeration)))
    {
      object castValue = converter.Convert(value);
    }            
  }
    
  stopwatch.Stop();
    
  Console.Out.WriteLine("Enumeration: IntEnumeration, Converter: DynamicMethod, Time: {0} ms", 
    stopwatch.ElapsedMilliseconds);
}
```

Well, I was going to repeat this all over the place for the numerous enumeration values I had, along with the two converters I've created (both were subclasses of `EnumConverter`). And I also wanted control over the number of times the loop iterated. Fast forward to the refactored method:

```c#
private static void TimeConversion(Type enumeration, EnumConverter converter, int iterations)
{
  Stopwatch stopwatch = Stopwatch.StartNew();

  for(int i = 0; i < iterations; i++)
  {
    foreach(object value in Enum.GetValues(enumeration))
    {
      object castValue = converter.Convert(value);
    }            
  }
    
  stopwatch.Stop();

  Console.Out.WriteLine("Enumeration: {0}, Converter: {1}, Time: {2} ms", 
    enumeration.Name, converter.GetType().Name, stopwatch.ElapsedMilliseconds);
}
```

The method doesn't control a lot; I control what the method does. This makes the method very flexible. I don't need umpteen versions of the method; one does the job for each combination I need:

```c#
Program.TimeConversion(typeof(IntEnumeration), new CastEnumConverter(), iterations);
Program.TimeConversion(typeof(IntEnumeration), new DynamicMethodEnumConverter(), iterations);
Program.TimeConversion(typeof(ByteEnumeration), new CastEnumConverter(), iterations);
Program.TimeConversion(typeof(ByteEnumeration), new DynamicMethodEnumConverter(), iterations);
Program.TimeConversion(typeof(ULongEnumeration), new CastEnumConverter(), iterations);
Program.TimeConversion(typeof(ULongEnumeration), new DynamicMethodEnumConverter(), iterations);
```

Again, this isn't rocket science. It just shows the more you can make your code flexible and adaptable, the more (ugh! dare I say it?) reuse you'll get out of it.

> Published: 07.18.2007 10:06:28 AM CST