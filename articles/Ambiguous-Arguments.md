---
title: Ambiguous Arguments
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Ambiguous Arguments

## What is Wrong With This Code?

Language: Java 

```java
public DatabaseResultSet executeQuery(Object theClass, String sql)
{
  //  Implementation code goes here.
}
```

When I was on a Java/CORBA project, I ended up needing to use a database. I was about to use the `java.sql` classes when one of the developers stated that I should use a custom Java package that wrapped these classes. As this project had gone through three years of development before I started working on it, a lot of utility classes had popped up here and there within the infrastructure, with their database utility classes being one of them. They had added these utility classes to "simplify" database access, but as you'll soon see, their design was anything but simple. 

The method I documented above, `executeQuery()`, was used to return a `DatabaseResultSet` object based off of the query specified in the `sql` parameter. It was part of a class called `Database`. Rather than show you the underlying implementation, I thought I would simply show you the signature to clearly demonstrate the problem at hand. Thankfully, I had the source code for the implementation; otherwise, I would not have known what to do. 

What do I mean? Well, what is the first parameter used for? Go ahead, take a look at the signature for a while. Can't tell? You can venture a guess, but your guess is as good as mine; Guessing is definitely not the answer when it comes to building robust systems. Its intended use is that you should pass in the this value when you call `executeQuery()`: 

```java
public void someMethod()
{
  String sql = "select count(*) from some_table";
  Database dbAccess = new Database();
  DatabaseResultSet countValue = dbAccess.executeQuery(this, sql);
}
```

When I examined the source code for `executeQuery()`, it's using `theClass` to determine which object called it for tracing purposes. It extracts the name of the object passed to it via reflection, and then stores that to a trace file. 

Unfortunately, there are a lot of things wrong with this approach. I could be in a static method, which rules out a this value as the following example demonstrates: 

```java
public static void someMethod()
{
  String sql = "select count(*) from some_table";
  Database dbAccess = new Database();
  //  The following line of code doesn't work.
  DatabaseResultSet countValue = dbAccess.executeQuery(this, sql);
}
```

Also, there's nothing to stop me from making a new object based off of the `Object` type and passing that in, like this: 

```java
public void someMethod()
{
  String sql = "select count(*) from some_table";
  Database dbAccess = new Database();
  Object thisFaker = new Object();
  DatabaseResultSet countValue = dbAccess.executeQuery(thisFaker, sql);
}
```

> Note: This was actually my first approach, as I had no idea what to do the first time I encountered this method. But I knew this was wrong, which made me investigate further.  

Or, I could pass in any object, totally confusing the resulting trace: 

```java
public void someMethod()
{
  String sql = "select count(*) from some_table";
  Database dbAccess = new Database();
  Person aPerson = new Person("Jason", "Bock");
  DatabaseResultSet countValue = dbAccess.executeQuery(aPerson, sql);
}
```

Obviously, this is an extremely poor design. Database should not force a client to pass in an arbitrary object. This design relies upon the concept of "programming by convention", which simply never works. Programming by convention means that all developers should use the same rules within the system, even though the language and/or tool doesn't enforce them (i.e. all interfaces should start with the letter "I"). However, since there's nothing in the tool to ensure a convention is enforced, it's too easy to circumvent it. The problems start when the developers rely on the convention. All it takes is for one person to forget about it during the course of the project, and debugging nightmares can easily set in. 

When you program by convention, you're usually doing something that you know won't be clear to future users of your classes and methods. So, you need solid documentation up front to state why the given design was chosen and how you should use it. Unfortunately, such documentation usually doesn't exist. Even if it does, the contract is usually too loose to enforce, leaving the possibility of incorrect use very high. 

Another small, yet important, problem is the naming of the parameter. It's typed as an `Object`, but its name is `theClass`. These are two entirely different things. A class is the blueprint for the object as objects are class instances, but here these concepts are mixed. The method is typed to accept an `Object`, but the name of the argument suggests that it actually needs the class information of the object and not the object itself (which is what the implementation of `executeQuery()` is doing). 

Furthermore, one of the advantages of OO programming is encapsulation. From the client's perspective, you shouldn't have to know what is going on within a method to use it. But in this case, I had to look at the implementation. Granted, good Javadoc for this class would have been helpful, but that was nowhere to be found in this case. 

One other odd design decision to note: `executeQuery()` doesn't raise an exception. So what happens if I pass in a malformed SQL statement (which happens often during development)? I have to call another method on `Database` called `succeeded()` to see if it worked or not. To me, this nullifies the use of exceptions in Java, and it gives the developer the opportunity to ignore an error. 

## Possible Solution

The first thing I would do is get rid of the first argument: 

```java
public DatabaseResultSet executeQuery(String sql)
{
  //  Implementation code goes here.
}
```

Granted, this requires you to revisit your code base and refactor as necessary. For example, if you have the following code: 

```java
public void someMethod()
{
  String sql = "select count(*) from some_table";
  Database dbAccess = new Database();
  DatabaseResultSet countValue = dbAccess.executeQuery(this, sql);
}
```

you'd need to alter it like this: 

```java
public void someMethod()
{
  String sql = "select count(*) from some_table";
  Database dbAccess = new Database();
  DatabaseResultSet countValue = dbAccess.executeQuery(sql);
}
```

However, I think this is necessary. The more and more I code, I am discovering that separating concerns in code leads to maintainable systems. I don't think `Database` should be responsible for tracing the call stack. And even if it wanted to, there are other ways to do it without requiring the client to pass in an arbitrary object (e.g. dynamic proxies). Finding and fixing all invocations of `executeQuery()` would not take too much time, even if the code base was large. 

The other change that I would make is open for debate: 

```java
public DatabaseResultSet executeQuery(String sql) throws
  MalformedSQLException, NoDatabaseConnectionException
{
  //  Implementation code goes here.
}
```

The method should raise typed exceptions as needed. However, since the code base was already using the idiom of testing a method for success (`succeeded()`), this change may be problematic. But, having typed exceptions forces the client to be aware of the fact that invoking this method can lead to errors that the method knows can occur. Having the client check a status value means you're using "programming by convention" again, and there's no way to enforce this. Granted, if you didn't check `succeeded()`, the return value may be `null`, and you'd eventually run into errors when you tried to read the result set. But using exceptions makes the design explicit. 

## Summary

1. Keep your clients in mind when defining a method. Make sure you've covered all possible scenarios.
2. When your design becomes an example of "programming by convention," refactor it. Chances are, you'll save time and money in the long run.

> Published: 05.30.2001