---
title: WCF, Client-Side Dynamic Proxies and Configuration
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WCF, Client-Side Dynamic Proxies and Configuration

Today I ran into another interesting feature of WCF. Let's say I'm making a client-side proxy dynamically through the ChannelFactory object:

```c#
using (ChannelFactory<IMyService> factory =
  new ChannelFactory<IMyService>())
{
  IMyService channel = factory.CreateChannel();
}
```

The client connection information was in the configuration file:

```xml
<system.serviceModel>
  <client>
    <endpoint address="net.msmq://localhost/private/SomeQueue"
      binding="netMsmqBinding"
      contract="ServiceDomain.IMyService" />
  </client>
</system.serviceModel>
```

I was getting an error: "The Address property on ChannelFactory.Endpoint was null. The ChannelFactory's Endpoint must have a valid Address specified." Which was really confusing the hell out of me, as the address was clearly stated in the configuration file. After a lot of futzing, I found an article, which finally showed me the error of my ways:

> The ChannelFactory<T> class shown in Listing 10 enables you to create a proxy on the fly. You need to provide the constructor with the endpoint-either the endpoint name from the config file or the binding and the address objects, or an endpoint object.

In other words, I needed to do this:

```c#
using (ChannelFactory<IMyService> factory =
  new ChannelFactory<IMyService>(string.Empty))
{
  IMyService channel = factory.CreateChannel();
}
```

... and all was well with the world again.

> Published: 01.22.2007 11:59:31 AM CST