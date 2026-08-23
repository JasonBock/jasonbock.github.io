---
title: Generics and Constraints
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Generics and Constraints

Before I left work today, I thought of an alteration to our design that would solve an issue we ran into. Tonight I confirmed it would work:

```c#
namespace GenericsAndConstraints
{
  public interface IBaseRequest { }

  public interface IBaseWorker<T> where T : IBaseRequest
  {
    void WorkIt(T request);
  }

  public interface ISubRequest : IBaseRequest { }

  public interface ISubWorker<T> : IBaseWorker<T> where T : ISubRequest { }

  public class SubWorker : ISubWorker<ISubRequest>
  {
    public void WorkIt(ISubRequest request) { }
  }
}
```

I was pretty sure it **would** work, but I had to do the Alt+B+R dance before I was sure.

> Published: 01.22.2007 07:36:37 PM CST