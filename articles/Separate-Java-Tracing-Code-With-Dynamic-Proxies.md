---
title: Separate Java Tracing Code With Dynamic Proxies
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Separate Java Tracing Code With Dynamic Proxies

## Introduction

During the evolution of a software project, it usually becomes necessary to implement some kind of tracing or logging framework to determine the cause of system defects. In general, this takes the form of a developer adding in statements to the code that log information to a file or the output stream. Although this works, it burdens the developer not only with adding in the tracing statements but also stripping the statements out when the system is released for production. In this article, I'll show an alternative solution in Java that uses dynamic proxies, a new addition to Java with the 1.3 release.

## A Typical Example

Let's say you've been given the following UML diagram and your job is to implement the design that models a person:

(2026 - sorry, the image has been lost)

Even if you're not familiar with Java, the implementation is pretty straightforward to follow. Here's the code for the `IPerson` interface:

```java
package com.viktor.person;

public interface IPerson
{
  public String getFirstName();
  public String getLastName();
  public int getAge();
  public void setFirstName(String firstName);
  public void setLastName(String lastName);
  public void setAge(int age);
}
```

As you can see, we're adding this interface to the `com.viktor.person` package, and we've defined the interface with the appropriate `getXXX()` and `setXXX()` methods. The implementation of `IPerson` involves storing the first and last name of the person along with his/her age (I'll only show the implementation for `getFirstName()` and `setFirstName()` for brevity's sake):

```java
package com.viktor.person;

public class Person implements IPerson
{
  private String firstName = null;
  private String lastName = null;
  private int age = 0;

  public Person(String firstName, String lastName, int age)
  {
    this.init(firstName, lastName, age);
  }

  public Person()
  {
    this.init("", "", 0);
  }

  private void init(String firstName, String lastName, int age)
  {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
  }

  //  Implementation of IPerson methods.
  public String getFirstName()
  {
    System.out.printlng("getFirstName was called.");
    return this.firstName;
  }

  public String setFirstName(String firstName)
  {
    System.out.printlng("setFirstName was called.");
    this.firstName = firstName;
  }

  //  Other method implementations go here.
}
```

So far, so good. To test our implementation, we can create a test class that ensures our `Person` implementation is solid:

```java
package com.viktor.logproxy;

import com.viktor.person.*;

public class TestProxies
{
  public static void main(String[] args)
  {
    IPerson basicPerson = new Person("Joe", "Smith", 25);
    basicPerson.setAge(35);
    basicPerson.setLastName("Quartz");
    System.out.println("In TestProxies, the name of the person is " + basicPerson.getFirstName() + " " + basicPerson.getLastName());
    System.out.println("In TestProxies, the age of the person is " + Integer.toString(basicPerson.getAge()));
  }
}
```

The name of the class, `TestProxies`, alludes to some of the work we'll do later on, but, for now, ignore the reference to the word "proxy." We create a new instance of the `Person` class and use this implementation of the `IPerson` interface to alter the information about Joe Smith. Once we've done our alterations, we print out the results.

This is where the problem lies - the tracing statements. We have tracing code compiled into our `Person` class so we know that certain methods are being invoked. Also, we've created a test class that has tracing code within it as well; the `Person` class is already doing this so this test class is unnecessary. However, if you didn't have access to the source code for the `Person` class, you may not know if it's doing any tracing.

What we'd like to be able to do is trace the class without editing the code needed to implement an interface. Peppering your business objects with `System.out.println()` may get the job done, but it can make for a source code maintenance nightmare (or at least source code obscfurcation). But if we could determine that a class was instantiated and grab the occurrence of method invocations along with some helpful information (e.g. argument values, timestamp of the method invocation, etc.) we may be able to pinpoint where the error is occurring. Furthermore, such a tracing strategy could apply to virtually any Java class that has ever been compiled (with a few caveats, as we'll see later on).

## Dynamic Proxies Explained

One of the additions that were made to Java in the 1.3 release of the language was the `Proxy` class to the `java.lang.reflect`, or Reflection, package. The `Proxy` class provides the developer with a way to create a class that implements a list of interfaces - this is known as a dynamic proxy class. An instance of this class is (you guessed it) a proxy instance, or a proxy object. The proxy object must be able to handle invocations of the interfaces initially given to create the proxy class - this obligation is fulfilled if the proxy object implements the `InvocationHandler` interface.

If this sounds confusing, a couple of diagrams may help in clarifying the proxy scenario. If you create a `Person` object like this:

```java
IPerson somePerson = new Person("Joe", "Smith", 55);
```

`IPerson` is referring to a `Person` instance:

(2026 - sorry, the image has been lost)

However, now let's create a dynamic proxy (assume that `ProxyObject` implements `InvocationHandler`):

```java
Object newPerson = new Person();
Class personClass = newPerson.getClass();
IPerson somePerson = (IPerson)Proxy.newProxyInstance(personClass.getClassLoader(),
personClass.getInterfaces(), 
new ProxyObject(newPerson));
```

The method `newProxyInstance()` creates the dynamic proxy object. The first two arguments use class information about the object to be wrapped - namely, what the class's class loader is and the list of interfaces that the class implements. The last argument is an `InvocationHandler` interface reference - I'll cover this later on in the article. Now, `IPerson` is actually referring to a proxy object:

(2026 - sorry, the image has been lost)

However, the client is oblivious to this. Since the proxy object implements `IPerson`, Java is perfectly happy with the current state of object references.

Granted, this may sound a bit odd the first time around. What use is there to create an object that does nothing but intercept and forward requests from a client? But that's the power of dynamic proxies. You can change the implementation of an object without the client's knowledge. You can alter the inputs, you can enhance the error handling, you can use your own home-grown version of object remoting - in short, you can control how the real object is accessed.

There are some limitations to proxy objects. For example, you can only create proxy objects for objects that implement interfaces. Therefore, if you try to create a proxy for a class, `Proxy` won't be able to give you a new proxy instance via `newProxyInstance()`. This design does enforce interface-based programming, but your current designs may not use interfaces as the point of communication between a client and the realization of that interface (i.e. the class that implements the interface). Also, if two or more interfaces define the exact same method, Java resolves the collision by invoking the method on the first interface that contains the method from the list generated by a class object's `getInterfaces()` method. There are some more technical issues with proxies, such as serialization, that go beyond the scope of this article, so please read the API documentation for more information.

## Implementing `LogProxyFactory`

To make this concrete, let's create a class called `LogProxyObject` that allows us to wrap objects with dynamic proxies. We'll keep the main class very thin - in fact, it will only have one static method called `logObject()`. Here's the code for `LogProxyObject`:

```java
package com.viktor.logproxy;

import java.lang.reflect.*;

public class LogProxyFactory
{
  public static Object logObject(Object obj)
  {
    //  This method creates a Proxy for the given object.
    Class argClass = obj.getClass();
    return Proxy.newProxyInstance(argClass.getClassLoader(),
      argClass.getInterfaces(),
      new LogProxy(obj));
  }
}
```

As you can see, `logObject()` has one argument, `obj`. This object reference is used to create a proxy object via the `newProxyInstance()` method on `Proxy`. However, you'll notice that the last argument to `newProxyInstance()` is a reference to an object of type `LogProxy`. `LogProxy` is a class I created that implements the `InvocationHandler` interface. Here's the code to `LogProxy` - I'll explain how it works after you spend some time perusing it:

```java
package com.viktor.logproxy;

import java.lang.reflect.*;

class LogProxy implements InvocationHandler
{
  private Object obj;

  public LogProxy(Object obj)
  {
    this.obj = obj;
  }

  //  These three logXXX() methods handle logging information.
  protected void logBeforeInvocation(Method m, Object[] args)
  {
  }

  protected void logErrorAfterInvocation(Method m, String errorMessage)
  {
  }

  protected void logAfterInvocation(Method m, Object returnValue)
  {
  }

  //  Implementation of InvocationHandler interface.
  public Object invoke(Object proxy, Method m, Object[] args) throws Throwable
  {
    Object result = null;
    try
    {
      this.logBeforeInvocation(m, args);
      result = m.invoke(obj, args);
    }
    catch(InvocationTargetException ite)
    {
      //  Log this error.
      this.logErrorAfterInvocation(m, ite.getMessage());
      throw ite.getTargetException();
    }
    catch(Exception e)
    {
      //  Log this error.
      this.logErrorAfterInvocation(m, e.getMessage());
      throw new RuntimeException("Unexpected target exception: " + e.getMessage());
    }
    finally
    {
      //  Here we can do post-processing.
      this.logAfterInvocation(m, result);
    }
    return result;
  }
}
```

We store the object that we think has been wrapped by a dynamic proxy in the constructor. I say, "we think," because a class in the `com.viktor.logproxy` package can create instances of `LogProxy` independent of a dynamic proxy. I could have added `LogProxy` as an inner class of `LogProxy`, but this would have left me with the choice of either making `LogProxyFactory` a singleton or I would've forced the client to make a new instance of `LogProxyFactory`. I wanted to keep it as simple as possible, so I made `LogProxy` its own class. There's really no reason that a client would add their class to this package just to create a `LogProxy` class instance; the true power of this class is when `LogProxyFactory` uses it.

Anyway, back to the description of `LogProxy`. Since we're implementing `InvocationHandler`, we need to implement `invoke()`. This is the essence of `LogProxy`, because `invoke()` will be called whenever the client calls a method on the dynamic proxy. This is where we can log method invocation information, which is what `logBeforeInvocation`, `logErrorAfterInvocation`, and `logAfterInvocation` are for. I've left them protected as I want other developers to have the ability to subclass `LogProxy` and implement the `logXXX()` methods as they see fit. To demonstrate how these methods work, I'll simply add `System.out.println()` statements in like this:

```java
protected void logBeforeInvocation(Method m, Object[] args)
{
  System.out.println(obj.getClass().getName() + "." + m.getName() + " will be invoked.");
}
```

To test this out, let's create a small test class called `TestProxies`:

```java
package com.viktor.logproxy.test;

import com.viktor.logproxy.*;
import com.viktor.person.*;
import java.lang.*;

public class TestProxies
{
  public static void main(String[] args)
  {
    try
    {	
      IPerson basicPerson = new Person("Joe", "Smith", 25);
      basicPerson.setAge(35);
      basicPerson.setLastName("Quartz");
      System.out.println("In TestProxies, the name of the person is " + basicPerson.getFirstName() + " " + basicPerson.getLastName());
      System.out.println("In TestProxies, the age of the person is " + Integer.toString(basicPerson.getAge()));

      //  Do it with a proxy.
      IPerson tracePerson = (IPerson)LogProxyFactory.logObject(new Person("Jason", "Bock", 29));
      tracePerson.setAge(30);
      tracePerson.setFirstName("Elizabeth");
      System.out.println("In TestProxies, the name of the person is " + tracePerson.getFirstName() + " " + tracePerson.getLastName());
      System.out.println("In TestProxies, the age of the person is " + Integer.toString(tracePerson.getAge()));
    }
    catch(Exception e)
    {
      e.printStackTrace(System.out);
    }
  }
}
```

All we're doing is creating two `Person` objects, but the second one is wrapped by our `LogProxy` object. If you run this test, you should get the following output (assuming that you removed the `System.out.println()` statements from the `Person` class):

```
In TestProxies, the name of the person is Joe Quartz
In TestProxies, the age of the person is 35
com.viktor.person.Person.setAge will be invoked.
com.viktor.person.Person.setAge - invocation complete, return value is null
com.viktor.person.Person.setFirstName will be invoked.
com.viktor.person.Person.setFirstName - invocation complete, return value is null
com.viktor.person.Person.getFirstName will be invoked.
com.viktor.person.Person.getFirstName - invocation complete, return value is Elizabeth
com.viktor.person.Person.getLastName will be invoked.
com.viktor.person.Person.getLastName - invocation complete, return value is Bock
In TestProxies, the name of the person is Elizabeth Bock
com.viktor.person.Person.getAge will be invoked.
com.viktor.person.Person.getAge - invocation complete, return value is 30
In TestProxies, the age of the person is 30
Process Exit...
```

Note that the calls to `basicPerson` were not traced, but the calls to `person` were.

## Fine-Grain Tracing

Now we have a way to capture when a method was invoked and what the outcome of that invocation was. However, we can do even better than this. You may have noticed that we automatically create a proxy whenever we call `logObject()` on `LogProxyFactory`. In a way, we really haven't solved a lot, because we're still requiring the client to turn the trace log on or off by either adding the call to `logObject()` or to remove it. What would be better for the client is to just call `logObject()` whenever a new object is created, and use some kind of configuration mechanism separate from the code that would determine if the object should be wrapped by a dynamic proxy.

Therefore, to completely remove the knowledge of creating a proxy from the client code, we'll create an XML file that will contain a list of classes that should be wrapped by a proxy and which methods of that class should be traced. So if we assume that we change the implementation of `logObject()` to perform this lookup into the XML file, we can always register new objects with LogProxyFactory:

```java
IPerson somePerson = (IPerson)LogProxyFactory.logObject(new Person("Joe", "Smith", 29));
SomePerson.setAge(44);
```

Depending upon the configuration of the XML file, the invocation of `setAge()` may be logged, or it might not be logged, or some other action may take place. In this case, the client doesn't care whether it is or isn't. It just creates a `Person` object and lets `LogProxyFactory` do what it should.

So how do we do this? First, let's create the layout for the XML file. The root element is called `Classes`, which contains `Class` elements that represent the classes we want to log instances of. Each `Class` element will contain a name attribute, which will be a fully qualified Java class name. Each `Class` element will also contain a `Methods` element, which will contain `Method` elements that represent all of the public methods that we want to log. The `Method` element will have two attributes - `name` and `argumentSignature`. `name` is simply the name of the element; `argumentSignature` is a tokenized string of each type name that the method contains separated by the character "%". If the method doesn't have any arguments, this will be an empty string. Therefore, if we were logging a method that had two `String` arguments, `argumentSignature` would equal `java.lang.String%java.lang.String`.

Here's an example of an XML tracing file that instructs `LogProxyObject` to only trace `getFirstName()` and `setLastName()` method invocations on a `Person` class:

```xml
<?xml version="1.0"?>
<Classes>
  <Class name="com.viktor.person.Person">
    <Method name="getFirstName" argumentSignature=""/>
    <Method name="setLastName" argumentSignature="java.lang.String"/>
  </Class>
</Classes>
```

Of course, we should create a schema file to make sure our configuration file is valid, but I'll leave that as an exercise for the reader.

Now that we have our XML file defined, we need to incorporate it into our tracing framework. `LogProxyFactory` needs to know where the file is so it can load it into a DOM for XPath searches, and it also needs to know if updates have been made to the file since the information was last used. The first part is handled by using the `library` path property from the `System` properties object. The second part simply looks at the last time the file was modified and compares that to the last modified time value that was cached when the file was initially loaded. Here's what the code looks like:

```java
if(xmlConfig == null)
{
  xmlConfig = new File(System.getProperty(jvmPathKey) + System.getProperty(fileSepKey) + xmlLog);
}
else
{
  //  Check to see if the file has changed 
  //  since we first read it.
  File xmlConfigCurr = new File(System.getProperty(jvmPathKey) + System.getProperty(fileSepKey) + xmlLog);
  if(xmlConfigCurr.lastModified() > lastModified)
  {
    xmlConfig = xmlConfigCurr;
  }
}
```

Now we need to see if the class of the object that we were given is in the XML file. This is handled by a simple XPath query (written in pseudo-Java code):

```
"/Classes/Class[@name=\"" + obj.getClass().getName() + "\"]"
```

If the class is in the DOM, then we need to get a list of the methods that need to be traced by the `LogProxy` instance. We do this by concatenating the value of `name` and `argumentSignature` (which will uniquely identify the method) and putting all of those values into a set. This set needs to be passed to the `LogProxy` instance on construction, so we need to create a new constructor:

```java
public LogProxy(Object obj, Set methods)
{
  this.obj = obj;
  this.methods = methods;
}
```

Once this is done, all `LogProxy` needs to do is check the set every time `invoke()` is called. If the method is in the set, tracing is performed.

Note that even though `LogProxy` doesn't know how `LogProxyFactory` gets the method list, it still needs to know how `LogProxyFactory` will format the method signature. An even better design would be to create some kind of `Method` object that is passed rather than a set object. This object would allow the client to find out the name of the object (i.e. the method name) and the list of argument signatures in a type-safe manner; I'll leave that as an exercise for the interested reader.

## Proxies and Interfaces Revisited - Automated Tracing

Before I finish, I'd like to talk about the possibility of completely hiding the use of `LogProxyFactory` - i.e. you would never need to register an object to create a dynamic proxy. As it stands, even though our tracing environment is completely configurable, you still have to register objects with `LogProxyFactory` via the `logObject()` method. This extra step may or may not be an inconvenience for you, and that's understandable. Even though I think the architecture I've presented is straightforward and unobtrustive, I still don't like the fact that I have to call `logObject()` every time I make a new object. The ultimate solution would be to somehow hook the creation of an object such that a client would never have to do object registration.

I tried looking into the Java API to see if there was a way to do this. I eventually came up with this possibility. If I could create a specialized proxy for a given class loader, then I could catch when the `loadClass()` method was called on the `ClassLoader` object. I would then wrap the returned `Class` object with another proxy. This specialized proxy would watch for `newInstance()` invocations, and wrap the resultant `Object` with our familiar `LogProxy` object. There is a catch with the reflection API, namely that you can use a `Constructor` object to create a new object, but I could also watch for `Constructor` objects where the `newInstance()` method has been invoked.

Therefore, all the client has to do is register class loaders. In most cases, you'll only be dealing with the system class loader, so all the client would need to do is add the following one line of code to their system initialization routine:

```java
LogProxyObject.getInstance().addClassLoader(ClassLoader.getSystemClassLoader());
```

And you're done!

This sounds great, and I'd love to show you the solution in code. But I won't go into the implementation, because it doesn't work. Recall that you can only wrap objects with dynamic proxies that implement interfaces. `ClassLoader` doesn't implement an interface, so the `Proxy` object can't create a dynamic proxy for `ClassLoader`. Wrapping a `ClassLoader` object with an interface doesn't help either, because the calls that are going to the default class loader will go directly to the class itself and not to the wrapping interface.

Basically, with most of the Java classes, you're stuck. Very few Java classes are implementations of an interface; the only ones that aren't are the Collection APIs. Therefore, you won't be able to catch invocations of the String methods. This is somewhat disappointing as I find that an API based on interfaces is a much cleaner design.

I'm not one to give up so easily with the automated tracing idea, but at the time of this article's publication, I don't know how to hook the object creation event. If anyone has a solution for this, please let me know and I'll put an addendum to this article (giving credit where credit is due, of course!). One possibility is using the Java Debug API, but a cursory glance didn't seem appealing since the API is based on classes and not on interfaces.

## Conclusion

In this article, we took a look at what dynamic proxies were and how we could use their capabilities to separate logging responsibilities from both the client and the objects that it uses. I hope this article has demonstrated how useful dynamic proxies can be. Experiment with the `Proxy` class - there are far more applications of this new Java class than what has been demonstrated in this article.

> Published: 08.22.1999 12:00:00 AM CST