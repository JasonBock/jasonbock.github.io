---
title: Remoting Configuration Problem
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Remoting Configuration Problem

I've run into a small issue with Remoting. Hopefully someone can shed some light on this. I have a WinService that hosts two Remoted objects - its configuration file has the following elements:

```xml
<system.runtime.remoting>
  <application>
    <channels>
      <channel ref="tcp" port="5050" >
        <serverProviders>    
          <formatter ref="binary" typeFilterLevel="Full" />
        </serverProviders>    
        <clientProviders>
          <formatter ref="binary" />
        </clientProviders>
      </channel>
    </channels>
    <service>
      <wellknown mode="Singleton" 
        type="ServerNotification, Auditing" 
        objectUri="ServerNotification.soap"/>
      <wellknown mode="Singleton" 
        type="AdministrationService, AdministrationService" 
        objectUri="AdministrationService.soap"/>
      </service>
  </application>
</system.runtime.remoting>
```

I call `RemotingConfiguration.Configure()` and then I call `RemotingServices.Marshal()`. All is well on the server side. The problem is on the client side. When I try to hook up and invoke methods on both objects, I get the following exception:

```
System.Net.Sockets.SocketException: Only one usage of each socket address (protocol/network address/port) is normally permitted
```

This makes sense as both are listening on port 5050. But I can't figure out how I can set up via a .config file that each object will be listening on separate ports. I know I can do this via code via `RegisterWellKnowServiceType()`, but I want to do this via configuration. Is this possible, and if so, how is it done? Feel free to leave a comment if you know the answer - thanks in advance.

> Published: 02.14.2005 11:46:19 AM CST