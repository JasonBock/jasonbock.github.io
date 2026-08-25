---
title: Getting the Configuration Name for a Custom Section Handler
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Getting the Configuration Name for a Custom Section Handler

A while ago, I discussed a design technique to package a custom configuration section handler's information into a singleton. Recently I had a discussion with a developer where we started talking about section handler names in the configuration file and it got me thinking about a problem with that singleton technique. Usually with custom section handlers, the host (i.e. the EXE) names the section handler with a name that it knows about, and uses that name when it calls `ConfigurationSettings.GetConfig()`. However, with the singleton technique, the class now has the responsibility of knowing what the section handler name is [1]:

```c#
public static AnotherConfiguredSectionHandler Instance
{
  get
  {
    if(handler == null)
    {
      handler = ConfigurationSettings.GetConfig("MyWellKnownName") 
        as AnotherConfiguredSectionHandler;
    }

    return handler;
  }
}
```

The issue is that it's possible that the configuration file doesn't use `MyWellKnowName` as the `<section>` element name, so all sorts of badness could ensure. However, I found a way around this by loading the current configuration information into an `XmlDocument` and finding out the name of the section handler via the handler's type name. This helper class is called `ConfigurationSectionNameFinder`, and as the following code shows, it's no longer necessary for any code to know a well-known name - it's all based on the type name of the class that implements `IConfigurationSectionHandler`:

```c#
public static AnotherConfiguredSectionHandler Instance
{
  get
  {
    if(handler == null)
    {
      string configurationName = ConfigurationSectionNameFinder.Find(
        typeof(AnotherConfiguredSectionHandler));
      handler = ConfigurationSettings.GetConfig(configurationName) 
        as AnotherConfiguredSectionHandler;
    }

    return handler;
  }
}
```

Please me know if you have any problems with it. Enjoy!

[1] Of course, the EXE could pass in the section name to the section handler, but as I demonstrate with this technique the section name becomes a non-issue, and by having everything in the section handler, the client still only needs to call .Instance and all is (still!) well with the world.

> Published: 09.02.2005 05:42:50 PM CST