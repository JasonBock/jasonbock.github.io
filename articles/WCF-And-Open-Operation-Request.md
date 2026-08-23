---
title: WCF and "Open" Operation Request
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WCF and "Open" Operation Request

I wanted to write a contract with an operation that had the following signature:

```c#
[ServiceContract]
public interface ITakeAnythingService
{
  [OperationContract(IsOneWay = true)]
  void TakeThis(object message);
}
```

The idea was to allow a client to post anything over to the server. Of course, this ended being a very bad idea, as I started getting all sorts of exception of the `System.ServiceModel.CommunicationException` kind. The error message was pretty verbose and actually contained information that started me down an alternative, successful path:

> Add any types not known statically to the list of known types - for example, by using the KnownTypeAttribute attribute or by adding them to the list of known types passed to DataContractSerializer.

Hmmmm ... `KnownTypeAttribute`. What's that? Well, to make a long story short, [this post](https://web.archive.org/web/20160201220422/http://blogs.msdn.com/b/sowmy/archive/2006/06/06/618877.aspx) was very helpful in describing the nuances of `KnownTypeAttribute`, along with how you can configure known types. What I ended up doing is changing the contract like so:

```c#
[ServiceContract]
public interface ITakeAnythingService
{
  [OperationContract(IsOneWay = true)]
  void TakeThis(Request message);
}
```

Where `Request` was defined like this:

```c#
public abstract class Request { }
```

Then I added the following configuration information:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.runtime.serialization>
    <dataContractSerializer>
      <declaredTypes>
        <add type="ServiceDomain.Messages.Request, ServiceDomain">
          <knownType type="ServiceDomain.Messages.AKnownRequest, ServiceDomain"/>
          <knownType type="ServiceDomain.Messages.AnotherKnownRequest, ServiceDomain"/>
        </add>
      </declaredTypes>
    </dataContractSerializer>
  </system.runtime.serialization>
</configuration>
```

The key is the `declaredTypes` element. You can list all of the known types that derive from the base type that your service can handle. What I like about this is that it forces the client to make a request that derives from a known base type on the server side, and the server must know about this new type before it'll handle it. Usually I don't advocate an "open" design like this across the board. Messages should be carefully crafted and designed with a known structure upfront. That said, in this case I needed to make an exception to this rule-of-thumb as my service needed to be fairly open and configurable.

By the way, don't use `System.Object` as the "root" known type (i.e. the type defined in the `type` attribute of the `add` element). I don't remember what the error message was, but basically I wasn't allowed to do that. In retrospect, that was a good thing anyway as I was violating on of my core tenets of service-based design: one parameter in of a known base type and (if the operation is not one-way), one return value of a known base type out.

> Published: 01.22.2007 07:06:54 AM CST