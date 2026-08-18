---
title: Why I Add Context To Members
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Why I Add Context To Members

One of the coding rules I (try to) follow is to add some context in front of the member I'm using. For example, if it's an instance field, I'll do this:

```c#
this.firstName = "Jason";
```

If it's a static member like a method, the name of the class goes in front:

```c#
Customer customer = Customer.Create("Jason", "Bock");
```

I realize that the compiler is smart enough to figure out what you're really referencing, but from a readibility standpoint it makes it very evident where it's coming from. I just ran into a "real-world" example where not having this confused the hell out of me. On my current project I'm working on hardening the code base by getting rid of bad exception handling blocks. One of the classes is called `SecurityContext`, and it has an instance method called `IsAuthorized()`. OK, so I start digging around to see how it's being used in the code, and I find a piece of code that looks like this:

```c#
SecurityContext.IsAuthorized("Delete");
```

Huh? I thought `IsAuthorized()` was an instance method - how is this working?

The reason is that there's a static property on the class called `SecurityContext`, which returns an instance of `SecurityContext`. If the code would've been written like this:

```c#
TimeService.SecurityContext.IsAuthorized("Delete");
```

Then it would've been much more apparent what `SecurityContext` is and where it's coming from.

It's a small thing, and I generally lean towards less characters in code than more [1]. But qualifying where the member is coming from is a case where I've found it beneficial to add some characters.

[1] Another one is always adding curly braces for `if` blocks. I know you can get away with not having them if you only have one line of code after the `if` statement, but that's prone to maintenance issues.

> Published: 07.16.2008 10:09:12 AM CST