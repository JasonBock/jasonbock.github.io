---
title: Unit Testing and Thread Apartment States
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unit Testing and Thread Apartment States

A while back, I lamented about the differences between NUnit and TestDriven.NET in terms of setting the apartment state of the current thread. Today I ran into the same issue (again!), but for some reason my brain came up with a solution that should work for both harnesses:

```c#
[Test]
public void DoWork()
{
  Thread testThread = new Thread(new ThreadStart(this.MTADoWork));
  testThread.ApartmentState = ApartmentState.MTA;
  testThread.Start();
  testThread.Join();
}

private void MTADoWork()
{
  // Test implementation goes here...
}
```

It's not pretty, but at least it ensures that the test code will run with the desired apartment state.

> Published: 05.23.2005 03:15:28 PM CST