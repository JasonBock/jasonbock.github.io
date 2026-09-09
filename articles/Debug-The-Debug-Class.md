---
title: Debug The Debug Class
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Debug The Debug Class

## What is Wrong With This Code?

Language: Java 

```java
package com.somepackage;

public abstract class Debug 
{
  public static void println(Object callingObject, 
    int messageLevel,
    String message) 
  {
    if (callingObject != null) 
    {
      if (_debug && messageLevel >= _debugLevel) 
      {
        System.out.println(callingObject.getClass().getName()
          + ": (" + messageLevel + ") " + "\n\t" + message);
      }
    } 
    else 
    {
      if (_debug && messageLevel >= _debugLevel) 
      {
        System.out.println(messageLevel + ") " + "\n\t"
		  + message);
      }
    }
  }

  public static void setDebugLevel(int level) 
  {
    //_debugLevel = level;
        
    if (level != OVERRIDE) 
    {
      _debugLevel = HIGH;
    } 
    else 
    {
      _debugLevel = INFO;
    }

    System.out.println("debug level: " + level);
  }

  public static final int HIGH = 5;
  public static final int MEDIUM = 4;
  public static final int LOW = 3;
  public static final int WARNING = 2;
  public static final int INFO = 1;
  public static final int OVERRIDE = 99;

  private static int _debugLevel = MEDIUM;
  private static boolean _debug = true;
}
```

This class was used throughout a project to allow the developer to debug a project. I don't like the name of the class as it is a bit deceiving. It doesn't debug code; it's really a tracing mechanism for the coder. I also don't like the name of the function `println()`; I would have called it `traceMessage()`. I'll give the implementor credit as the preconditional check for the object being `null` is made, but notice that there is another preconditional check in both the `if` and `else` part of `println()` - that is, `_debug` must be `true` and the given message level must exceed or equal the current debugging level. This can be cleaned up a bit - I'll show how later on. I also would have made the output location dynamic. That is, you could change the output from a console window to a file. But my real beef is with `setDebugLevel()`. 

> Note: There are other alternatives to tracing in Java that can be less obtrusive than peppering your code with `Debug.println()` statements. You can use dynamic proxies to trace method calls - check out my web site for an article on how you can do this (it's called, "Separate Java Tracing Code With Dynamic Proxies"). If you use interfaces throughout your project, you can have an extremely flexible tracing framework that can be customized on the fly. Another option is AspectJ, which is based on the concept of aspect-oriented programming (AOP). This allows you to define aspects that will add pieces of code to your code base before the class compilations begins. This keeps your code clean of tracing statements.  

First, note that we have five debug levels defined as constants (i.e. static finals - they're constants to me). We also have one constant that is used to "override" a previous value. But if you logically trace through `setDebugLevel()`, you'll notice that we only have two options: 

1. If we pass in anything other than `OVERRIDE`, the level is set to `HIGH`.
2. Otherwise, the debug level is set to `INFO`.

Note that the level is originally set to `MEDIUM`. 

This is a prime example of code that compiles, but is completely worthless. For example, what if I want to set the level to `LOW`? I can't. And if I "override" it, I can't set it back to `MEDIUM`. You may as well get rid of `LOW` and `WARNING`, and quite possibly `MEDIUM` as that is never used again if `setDebugLevel()` is ever called. Also, it's perfectly valid that I pass in a value other than 1, 2, 3, 4, 5, or 99 (e.g. 42) when it's clear that these six values are the only "correct" values. 

It's also funny to see that there's a comment in the code that would set the internal value to the value given by the client. This is the way I would've done it, and as you'll see in a moment, this is how I "fix" the method. However, it was unclear to me why this was commented out and replaced with the stellar code you now see. In retrospect, I should've went through Visual Source Safe to determine the evolution of this method's implementation, but it's clear that someone decided that one line of code wasn't good enough. Finally, note that there is no `getDebugLevel()` method. If I want to know what the current level is, I'm out of luck. 

When I see code like this, it infuriates me. There is simply no excuse for something like this. This shows no thought on the part of the developer who created such a monstrosity. And the funny thing is, it's not that hard to make a simple tracing class. 

## Possible Solution 

```java
public abstract class Trace 
{
  public static void traceMessage(Object callingObject, 
    int messageLevel,
    String message) 
  {
    if(_trace && messageLevel >= _traceLevel)
    {
      if (callingObject != null) 
      {
        System.out.println(callingObject.getClass().getName()
          + ": (" + messageLevel + ") " + "\n\t" + message);
      } 
      else 
      {
        System.out.println("(" + messageLevel + ") " + "\n\t"
		  + message);
      }
    }
  }

  public static void setTraceLevel(int level) 
  {
    if(level >= INFO && level <= HIGH)
    {
      _traceLevel = level;
      System.out.println("Trace level has changed to "
        + _traceLevel);
    }
  }
    
  public static int getTraceLevel()
  {
    return _traceLevel;
  }

  public static final int HIGH = 5;
  public static final int MEDIUM = 4;
  public static final int LOW = 3;
  public static final int WARNING = 2;
  public static final int INFO = 1;

  private static int _traceLevel = MEDIUM;
  private static boolean _trace = true;
}
```

Changing the name of the class to `Trace` is arguably not needed, but I think it's more semantically correct than `Debug`. Since I made other changes to the class, we might as well go the whole nine yards. I felt that removing `OVERRIDE` was necessary as it really served no purpose; you're overriding the trace level whenever you call `setTraceLevel()` anyway. I added a `getTraceLevel()` method (although one can also argue that if the requirements don't call for it, don't add it. To borrow from Martin Fowler: "You Aren't Going to Need It"). I also changed the preconditional logic in `println()` (which is now called `traceMessage()`), and I added a preconditional check in `setTraceLevel()`. Since these changes are purely implementation-based, they don't affect any of the clients currently using the class. Again, given possible time constraints, changing class and method names may not be feasible. However, the underlying implementations, in my opinion, must change. 

One thing that I still don't like is the fact that you can only turn tracing on or off during compilation time. I think this should be dynamic - that is, you can toggle it while the system is running. There are ways to change this, but I'll leave that as an exercise to the reader. (Note that if you do tackle this execise, you'll also want to add a configuration mechanism in order to change the output location as well). 

One final idea that you can try to add into `Trace` is to create an `Exception` object in `traceMessage()` so the caller wouldn't have to pass its object reference to `traceMessage()` (as a side note, what happens if you call `traceMessage()` from a static method?). Just parse the stack frame in the `Exception` object and you'll have all the call stack information you need. In .NET, this is extremely trivial, as there are the `StackFrame` and `StackTrace` classes that simplify this approach. This functionality isn't necessary, though, and you need to weigh the elimination of an argument to the refactoring that would need to take place in the client along with the creation of an `Exception` object and the eventual stack parsing. 

Summary 

1. Before you make changes to code, see if the current implementation is satisfactory.
2. Logically trace through the code's execution before you release it to production.

> Published: 04.25.2001