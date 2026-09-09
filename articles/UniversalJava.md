---
title: UniversalJava
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# UniversalJava

Recently I worked on a very reflection-heavy Java system. To be fair, I like reflection and the project needed to use it for valid reasons, but when an entire system is based around it, you start tearing your hair out (and for me, that's not a pretty picture). I've come to love strongly-typed systems - in fact, Meyer's OOSC book has made me appreciate compile-time type checking even more than I did before. Therefore, having to use the `Class`, `Method`, and `Field` classes ad nauseum left me reeling. During one period of frustration, I created four Java classes and packaged them up as the `UniversalJava` package. I present the code here for your enjoyment (although if you laugh, you are either a programming geek or you think I'm a complete weirdo).

```java
package com.universal;

public interface UniversalInterface
{
  public Object[] universalMethod(String methodName, Object[] params) throws Exception;
}
```

```java
package com.universal;

public abstract class UniversalAbstractClass implements UniversalInterface
{
  private String className = null;

  public UniversalAbstractClass(String className)
  {
    this.className = className;
  }
}
```

```java
package com.universal;

public class UniversalClass extends UniversalAbstractClass
{
  public UniversalClass(String className)
  {
    super(className);
  }

  public Object[] universalMethod(String methodName, Object[] params) throws Exception
  {
    //  Use reflection and any other typeless run-time construct prone
    //  to exceptions of the most glamorous kind to figure out
    //  what should be done with this method invocation.
    return null;
  }
}
```

```java
package com.universal;

public class UniversalTest
{
  public static void main(String[] args)
  {
    try
    {
      UniversalClass uc = new UniversalClass("Customer");
      Object[] firstName = {"Hugh"};
      uc.universalMethod("setFirstName", firstName);
      Object[] lastName = {"Gass"};
      uc.universalMethod("setLastName", lastName);
    }
    catch(Exception e) {}
  }
}
```

Of course, `UniversalTest` does nothing. You need to subclass `UniversalClass` and add some hashtables such that `universalMethod` invocations can be verified.

Remember, this is all a joke. I didn't use this in the project, but it sure felt like it!

> Published: 09.15.2000 12:00:00 AM CST