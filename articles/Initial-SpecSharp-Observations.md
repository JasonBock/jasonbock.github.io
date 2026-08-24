---
title: Initial Spec# Observations
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Initial Spec# Observations

One of the languages I'm taking a look at for my talk at the Code Camp is Spec#. Design-by-contract is not a new concept to me - I learned all about it when I read the "Object Oriented Software Construction" book years ago, so it's been fairly simple to pick up on the Spec# concepts. However ... the system is kind of buggy right now. It didn't pick up on some contract errors I had purposely made in my code; it took a unload-and-reload solution dance to get the expected squiggly lines in VS [1]. Also, I tried to overload `ToString()`:

```c#
public override string ToString() { }
```

And the compiler gives me the following errors:

```
'SpecSharpPerson.Person.ToString()': cannot change return type when overriding inherited member 'object.ToString()'
'SpecSharpPerson.Person.ToString()' is not pure enough. It either overrides or implements 'object.ToString()' which is marked as 'Microsoft.Contracts.ConfinedAttribute'
```

O ... K. The same code in a C# class doesn't bark the same way, so I'm guessing it's a Spec# bug.

I'll keep digging into the Spec# world over the next month or so, and sum up my results at the camp (along with a couple of other languages I'm researching). There's a lot of concepts in Spec# I agree with, even if the tool itself has some wrinkles here and there.

[1] I know Spec# has a command-line compiler, and the bugs I'm seeing may be just VS integration issues, but the command line and I just don't get along.

> Published: 09.21.2006 08:22:48 AM CST