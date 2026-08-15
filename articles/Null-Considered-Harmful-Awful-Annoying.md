---
title: Null Considered ... Harmful? Awful? Annoying?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Null Considered ... Harmful? Awful? Annoying?

I heard about an upcoming presentation about Hoare's "1 billion dollar mistake". Now, I'm not a lawyer, so I don't know if his "confession" means he has to pay damages to developers (I kid, I kid!), but I was surprised that he thought the concept of null was such a bad, bad idea.

However, the more I thought about it, the more the idea resonanted with me.

I mean, I'm always decorating my methods like this:

```c#
public void DoSomethingAwesome(TimeMachine machine)
{
  if(machine == null)
  {
    throw new ArgumentNullException("machine");
  }
}
```

Nulls always seem like a hinderance. I really don't want to ever do anything to a null reference, because the program will crash and burn badly.

So can we really get rid of them in code?

We, we can ensure code crashes when it sees a null reference from a parameter reference. That's easy. But ... what about factory methods? Like this:

```c#
var customer = CustomerFactory.Find(1234);

if(customer != null)
{
  // Do something interesting with the customer...
}
```

This is something I see all the time. In some cases, if a factory method cannot produce whatever it's trying to return, it's an error and an exception should be thrown. But in other cases, you're trying to find one specific item, like a customer. So returning null seems like a natural choice if the customer can't be found - I'd personally find it weird if the method threw `CustomerNotFoundException`.

But maybe we can repurpose the method like this:

```c#
var customer = Customer.Find(1234);

if(customer.Count > 0)
{
   // Pull out the customer and do something with it...
}
```

In other words, you return a collection. This seems like a better approach. Either you find 0, 1 or more than one. This also makes the API more consistent if you have overloads of the method that could return more than 1 customer:

```c#
var customer = Customer.Find("Joe", "Smith");

if(customer.Count > 0)
{
  // Pull out the customers and do something with them...
}
```

The convention with returning collections in .NET is that they should always be non-null, and the assumption would be that the collection only contains non-null references.

Of course, one could argue that a null check might be cheaper from an execution perspective than it would be to create a collection-like object, return it, and read the `Count` property. That's hard to say without doing some performance analysis, and like any performance optimizations, don't do them prematurely and don't guess. I ran some very simple tests comparing the collection return vs. null return check, and the null check is faster by about 37%. But that's just one test, and frankly, compared the rest of a large system, is that really that big of a deal? I could run my collection test code 5 million times in just over 1 second, and the null test code in 0.68 seconds. The gain with a non-null API seems better in my opinion.

Overall, I'm still not convinced that nulls rank up there with the evilness that gotos are (I'm also half-joking here). But I have to admit, I do find them annoying. The more I code, the more I try to follow more modern conventions (e.g. immutability, composition over inheritance, etc.). And writing code that doesn't tolerate null is slowly working itself into my approach.

What do you think? Is null a huge mistake? Should it be banished? Or are there actually valid uses for that 4-letter word?

> Published: 01.23.2009 07:30:07 AM CST