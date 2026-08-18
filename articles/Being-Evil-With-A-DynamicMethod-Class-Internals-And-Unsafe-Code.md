---
title: Being Evil With a DynamicMethod, Class Internals, and Unsafe Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Being Evil With a DynamicMethod, Class Internals, and Unsafe Code

> *WARNING*: This approach is still not working correctly, so don't use it! This is a true hack-fest, and I'm close to saying, "it completely works!", but until it works cleanly don't throw it into production code.

I don't know why, but I like IL. Well, "like" isn't the correct word - it's not a language I'd like to program in every day - but if you want to do some truly cool/interesting/odd things in your code, `System.Reflection` and `System.Reflection.Emit` are your friends. Lately, I've really fallen in love with `DynamicMethod`. There's no reason anymore to write "slow" Reflection-based code. You figure out what you need to do with the Reflection API, harden that code path by gen-ing a `DynamicMethod`, and now you have fast, performant, dynamic code.

During my last dynamic coding adventure, a bunch of ideas started to click together very, very fast.

* Extension methods are really cool, but they're just syntatic sugar - you don't get access to the internals of the object you're passed.
* But wait a minute ... dynamic methods do get access to internals.
* Hmmm ... if I generate a `DynamicMethod` for a class, I can do whatever I want to its internals.
* Hold on ... didn't a bunch of people try to create the fastest `Reverse()` method for `String`?
* OK, let's dump `String` to a file using my add-in for Reflector and see what's going on.
* Interesting, there's a field called `m_firstChar`, and there are internal methods that use it in unsafe code to append and insert characters ... wait a minute!!

This is where I went on a coding spree to see if I could create a fast `Reverse()` extension method.

For starters, let's take a look at two well-known implementations, one that reverses the character array and one that uses unsafe code to reverse the contents:

```c#
public static string Reverse(this string @this)
{
  char[] characters = @this.ToCharArray();
  Array.Reverse(characters);
  return new string(characters);
}

public unsafe static string ReverseViaPointers(this string @this)
{
  string output = string.Copy(@this);

  fixed(char* s = output)
  {
    char t;

    for(int x = 0, y = output.Length - 1; x < y; x++, y--)
    {
      t = s[x];
      s[x] = s[y];
      s[y] = t;
    }
  }

  return output;
}
```

Now, both approaches have their merits and problems, but both have one fundamental issue: they require copying the contents of the string and returning a new string. For small strings, this won't be a big issue, but if you have a very large string you may run into some memory issues. Unfortunately, since the `string` class doesn't have a `Reverse()` method on it, these approaches are about the best you can do ... unless you actually access the string's characters in-memory!

Showing you the `ILGenerator`-based code isn't really going to help (you can get all the code here), so what I'll do is show you a mocked-up `FakeString` class that has the code I want to generate at run-time:

```c#
public sealed class FakeString
{
  private char m_firstChar;

  private unsafe void DoIt()
  {
    fixed(char* s = &this.m_firstChar)
    {
      int index = 0;
            
      for(int i = this.Length - 1; index < i; i--)
      {
        char t = s[index];
        s[index] = s[i];
        s[i] = t;
        index++;
      }
    }
  }
    
  public int Length
  {
    get;
    private set;
  }
}
```

I compiled this class, looked at it in Reflector, and used the IL view to create the `DynamicMethod`.

So how does it compare to the other two approaches?

```
Iterations: 500000
String length: 80
Reverse via array copy: 2.5360865 seconds
Reverse via pointers: 2.3396009 seconds
Reverse via dynamic method: 2.3899677 seconds

Iterations: 500000
String length: 800
Reverse via array copy: 3.7658830 seconds
Reverse via pointers: 3.3728144 seconds
Reverse via dynamic method: 3.3586768 seconds

Iterations: 100000
String length: 8000
Reverse via array copy: 4.6696644 seconds
Reverse via pointers: 2.8743664 seconds
Reverse via dynamic method: 2.6584674 seconds

Iterations: 50000
String length: 80000
Reverse via array copy: 35.1555896 seconds
Reverse via pointers: 26.8925483 seconds
Reverse via dynamic method: 17.6794193 seconds

Iterations: 5000
String length: 800000
Reverse via array copy: 39.6807557 seconds
Reverse via pointers: 28.4833161 seconds
Reverse via dynamic method: 18.0042209 seconds

Iterations: 500
String length: 8000000
Reverse via array copy: 43.5914570 seconds
Reverse via pointers: 28.4863330 seconds
Reverse via dynamic method: 17.9445150 seconds
```

For small strings, it's not noticable. But as the strings get bigger, the array-copy approach degrades, which both the pointer and dynamic memory approach level off. However, since the dynamic approach doesn't copy the string, it's the fastest approach of all.

There's one huge caveat with this approach, though. Something with the way I'm accessing and using the string's contents isn't good. Here's what I saw when I ran some peformance tests on the code. Here's how things look with the first iteration:

![Reverse - First Iteration](https://jasonbock.net/images/Reverse-First-Iteration.png "Reverse - First Iteration")

Now, when I come through the loop the second, the local string variable value should be null again...right? Wrong!

![Reverse - Second Iteration](https://jasonbock.net/images/Reverse-Second-Iteration.png "Reverse - Second Iteration")

It's actually the reversed value from the first time around! But the real issue is what happens after the assignment:

![Reverse - Real Issue](https://jasonbock.net/images/Reverse-Real-Issue.png "Reverse - Real Issue")

D'oh!

This is a good illustration why speluking around in code that you don't own can be very dangerous. There may be an easy fix to what I'm doing terribly wrong, but even if figure it out, I'm dependent on how string is implemented in .NET 3.5. If that changes in any way in the future, my approach will fall apart again. Furthermore, I'm just flat-out going against the design of the string class. Strings are immutable. You don't change a string; you get a new string with the changes contained within it.

At the end of the day, this was a fun exercise, because it shows how you can get an extension method to act just a method on the object with access to everything inside of it. In a way, this feels like monkeypatching, although all I'm doing here is adding a method; I'm not changing implementations or removing them. All you need is a shim method using `DynamicMethod` and you're good to go ... just make sure you thoroughly test it!

> Published: 07.17.2008 04:45:20 PM CST