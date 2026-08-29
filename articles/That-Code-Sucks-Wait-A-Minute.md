---
title: That Code Sucks! Wait a Minute ...
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# That Code Sucks! Wait a Minute ...

Right now I'm typing in a most awkward position. Virtually all of our furniture is tucked away in the unfurnished part of our lower level, so I really don't have anywhere to sit, so I'm lying on the floor in the living room, typing away. But I ran into some code during my recent investigations of MVC and ASP.NET that, at first glance, looked like some really bad code. However, after I tested it out, it's not that bad at all; in fact, it contains a really cool configuration idea (although there's still one tweak I would do to it). Here it is:

```c#
using System;
using System.Web;
using System.Xml;
using System.Configuration;
using System.Collections.Specialized;

public class UrlMap : IConfigurationSectionHandler 
{
  private readonly NameValueCollection _commands = new NameValueCollection();

  public const string SECTION_NAME="controller.mapping";

  public static UrlMap SoleInstance 
  {
    get {return (UrlMap) ConfigurationSettings.GetConfig(SECTION_NAME);}
  }

  object IConfigurationSectionHandler.Create(object parent,object configContext, XmlNode section) 
  {
    return (object) new UrlMap(parent,configContext, section);   
  }

  private UrlMap() {/*no-op*/}

  public UrlMap(object parent,object configContext, XmlNode section) 
  {
    try 
    {
      XmlElement entriesElement = section["entries"];
      foreach(XmlElement element in entriesElement) 
      {
        _commands.Add(element.Attributes["key"].Value,element.Attributes["url"].Value);
      }
    } 
    catch (Exception ex) 
    {
      throw new ConfigurationException("Error while parsing configuration section.",ex,section);
    }
  }   
   
  public NameValueCollection Map
  {
    get { return _commands; }
  }
}
```

Now, my first impression was not good. Here were my issues:

* Why does this code have a public constructor that has the exact same parameters as the `Create()`? Why not just set the fields in an instance of `UrlMap` inside the interface implementation of `Create()`?
* `SoleInstance`. Dude, it's `Instance` - get the pattern right. Gosh!
* And this really isn't a singleton, because a new `UrlMap` is made **every single time** `SoleInstance` is invoked. That's bad.
* To save the best for last ... why is the no-argument constructor `private`?! How in the world is `ConfigurationSettings` going to create an instance of this class? Is that why the author made that other constructor? Is there some magic juju going on that I didn't know about in `ConfigurationSettings`?

Needless to say, I was a little miffed. This seemed like code that was awkward at best. I really didn't expect it to work, but to make sure I gave the code its' fair day in court I wrote a test for it and ran it (assume I have a section in my configuration file for the test that is given in the article I mentioned before):

```c#
using NUnit.Framework;
using System;

namespace FrontControllerConfiguration
{
  [TestFixture]
  public sealed class ConfigurationTests
  {
    public ConfigurationTests() : base() {}

    [Test]
    public void RunConfiguration()
    {
      Assert.AreEqual(2, UrlMap.Instance.Map.Count, "The count is incorrrect.");
    }
  }
}
```

Yeah, it succeeded. Wow.

What suprised me is that `ConfigurationSettings` was able to make an instance of `UrlMap`. What it does is it calls the no-argument constructor on `UrlMap`. It's doing this via `Activator.CreateInstance(type, bool)`, where the second parameter is `true` so it doesn't care if it's `public` or non-`public`. Then it calls `Code` via the interface implementation. That creates another instance of `UrlMap` (something I still don't like) and it returns that value.

I like this idea because the configuration information is all packaged up in one class. I would still make some changes, though:

```c#
using System;
using System.Web;
using System.Xml;
using System.Configuration;
using System.Collections.Specialized;

public class UrlMap : IConfigurationSectionHandler 
{
  public const string SectionName ="controller.mapping";

  private static UrlMap instance = null;
  private readonly NameValueCollection commands = new NameValueCollection();
    
  public static UrlMap Instance 
  {
    get 
    {
      if(UrlMap.instance == null)
      {
        UrlMap.instance = ConfigurationSettings.GetConfig(SectionName) as UrlMap;
      }

      return UrlMap.instance;
    }
  }

  object IConfigurationSectionHandler.Create(object parent,object configContext, XmlNode section) 
  {
    try 
    {
      XmlElement entriesElement = section["entries"];

      foreach(XmlElement element in entriesElement) 
      {
        this.commands.Add(element.Attributes["key"].Value,element.Attributes["url"].Value);
      }
    } 
    catch (Exception ex) 
    {
      throw new ConfigurationException("Error while parsing configuration section.", ex, section);
    }

    return this;
  }

  private UrlMap() : base() {}

  public NameValueCollection Map
  {
    get 
    { 
      return this.commands; 
    }
  }
}
```

I return this in `Create()`, rathen than making a new `UrlMap` object. I also have a "true" singleton - well, at least for this `AppDomain`.

So ... what's the moral of the story? A long time ago, I would've posted something right away about how sucky this code is. Maybe with age I'm learning not to be so quick on the trigger. I have to admit this code still annoyed me (especially since I couldn't find a working example of this code or any code in the article for that matter available for download) and it bugged me enough to dive into it and see if my hunches were correct. But I made sure I had some facts behind my thoughts first before broadcasting them. I would've looked really stupid saying, "there's no way this can be done because the constructor is `private`!!", because I was assuming `ConfigurationSettings` required a public no-argument constuctor. But ... it appears that it does! At least now that I know this I can use this idea in future project as the idea of having the configuration stuff contained from one starting location appeals to me - it'll keep the code base clean.

> Published: 07.08.2005 10:14:02 PM CST