---
title: I Want Intentionalsense
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Want Intentionalsense

Today I realized that Intelliense is just not good enough anymore. I want Intentionalsense. You know what I mean. Not some refactoring tool that can do a rename for me. Oh, no. I want something that knows **exactly** what I'm going to do next, and then it just does it. Wouldn't that kind of add-in to a code editor just **rock**?! Like, if I type:

```c#
public class Cust
```

It automatically does this:

```c#
public class Customer
{
  public string firstName;
  public string lastName;

  public Customer(string firstName, string lastName) : this()
  {
    if(firstName == null)
    {
      throw new ArgumentNullException("firstName");
    }

    if(lastName == null)
    {
      throw new ArgumentNullException("lastName");
    }

    this.firstName = firstName;
    this.lastName = lastName;
  }

  public string FirstName
  {
    get
    {
      return this.firstName;
    }
  }

  public string LastName
  {
    get
    {
      return this.lastName;
    }
  }
}
```

Or, better yet, it would've realized when I was trying to add session state to my mock `HttpContext`, it would've figured out how to do it ... all on its own. Think of all the time we could save! The tool would sense what our coding intentions are, and then do them perfectly.

Sometimes, ya gotta dream big dreams ...

> Published: 09.07.2005 10:40:58 PM CST