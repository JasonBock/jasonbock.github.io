---
title: Using Attributes to Name Resources in Assemblies
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using Attributes to Name Resources in Assemblies

While I've pretty much written off (no pun intended) writing another book (at least for a while), I would not mind doing a 2nd edition of the "Applied .NET Attributes" book [1]. I'm happy with the results, but there's some sections I'd like to clean up, and I'd like to add more material, and I'd like to clarify a couple of points, etc., etc. So far, no word has come back from the publisher to do a 2nd edition and I really, **really** doubt it'll ever happen, so I'm pretty much forced to use this blog to talk about some of the aforementioned deltas.

One aspect about attributes that people struggle with is, "just when do I use them?" I tried to answer that question in Chapter 1, but it's not an easy question, because in my mind there is no obvious answer. That said, what I've found over the years developing in .NET is that I'll use an attribute to add "seasoning" to the code base. That is, I don't find a lot of spots to use an attribute, but when I do I'm glad the support for custom metadata in .NET is there. Here's one example that I just ran into at a client [2]. Let's say you've added a bunch of resource files to your assembly that contains information about images, strings, and other data elements that you want to look up in the application via `ResourceManager`. The names of the two resource files are:

* StringStuff.resx
* ImagesGalore.resx

As you can image, StringStuff.resx contains string information, and ImagesGalore.resx contains all of your porn. Now, to simplify the lookup of these resources, you create a class called `ResourceService` that can be extended for different resource types, like `StringResourceService` and `ImageResourceService`. In other words, you're creating a component that will be used within a framework by other developers, so you're expecting the resources to have specific names so you can look them up. However, trying to generalize the lookup of resources is a bit problematic when using VS .NET. The resource file information after the project is compiled will end up as an assembly resource with a name that follows the pattern: {DefaultNamespace}.{ResourceFileName}, where {DefaultNamespace} is the *Default Namespace* project property. For example, the StringStuff.resx information will be named `ResourceServicesTests.StringStuff`, assuming `ResourceServicesTests` is the *Default Namespace* value. This makes things hard because I don't want a client of my resource services to tell me the name of the resource to look up; I want it to be pretty painless - something like this:

```c#
public void GetString()
{
  StringResourceService service = new StringResourceService();
  string resourceValue = service.GetString("TestData");
}
```

In this case,` StringResourceService` will look at the calling assembly and use it to find the string resources and get the string value for `TestData`. But, how does `StringResourceService` know which resource to load? If I force clients to name their string resource file with a well-known name like "Strings.resx", I still don't know what the "default namespace" is after compilation (remember, a namespace is just a fancy way to prepend the name of a class). But if I try to say to clients, "OK, guess what, blank out the *Default Namespace* project property in VS .NET so I have a chance in hell of loading your resource correctly", I know I'd get angry responses, because when you do that and you add a new class file to your project, you get this:

```c#
using System;

namespace 
{
  /// 

  /// Summary description for NewClass.
  /// 

  public class NewClass
  {
    public NewClass()
    {
      //
      // TODO: Add constructor logic here
      //
    }
  }
}
```

Ugly - the namespace is left blank, so now the developer has to go and fill it in correctly all the time. But, with the power of attributes, it become a simple problem to solve:

```c#
[assembly: ImageResourceName("ResourceServicesTests.ImagesGalore")]
[assembly: StringResourceName("ResourceServicesTests.StringStuff")]
```

What I've done is create an attribute called `ResourceNameAttribute` that can only be applied at the assembly level. This attribute's intent is to be extended for specific resource types (like `ImageResourceNameAttribute`). Furthermore, it can only be added once to an assembly. Here's how the base code looks:

```c#
using System;

namespace ResourceServices
{
  [AttributeUsage(AttributeTargets.Assembly, AllowMultiple=false)]
  public abstract class ResourceNameAttribute : Attribute
  {
    private string resourceName = string.Empty;

    protected internal ResourceNameAttribute() : base() {}

    public ResourceNameAttribute(string resourceName) : this()
    {
      if(resourceName == null)
      {
        throw new ArgumentNullException("resourceName");
      }

      this.resourceName = resourceName;
    }

    public string ResourceName
    {
      get
      {
        return this.resourceName;
      }
    }
  }
}
```

This is really nice because now I can assume that a client assembly can have at most one kind of resource to look up. This metadata lookup is done in `GetResourceName()` of `ResourceService`:

```c#
protected string GetResourceName(Assembly targetAssembly, 
  Type resourceNameAttributeType)
{
  string resourceName = null;

  if(targetAssembly.IsDefined(resourceNameAttributeType, false) == true)
  {
    ResourceNameAttribute resourceNameAttribute = 
      (ResourceNameAttribute)(targetAssembly.GetCustomAttributes(
        resourceNameAttributeType, false)[0]);
    resourceName = resourceNameAttribute.ResourceName;
  }
  else
  {
    resourceName = targetAssembly.GetName().Name;
  }

  return resourceName;
}
```

`Subclasses` of `ResourceService`, like `ImageResourceService`, look up the resource data using `GetResourceName()`:

```c#
public Image GetImage(Assembly targetAssembly, string resourceDataName)
{
  string resourceName = base.GetResourceName(targetAssembly,
    typeof(ImageResourceNameAttribute));
  ResourceManager resourceManager = 
    new ResourceManager(resourceName, targetAssembly);
  return (Image)resourceManager.GetObject(resourceDataName);
}
```

The bottom line is that I don't force the clients to change any project configuration or well-known file names. All they have to do is add a line of code for each resource set they have in their assembly - the resource services takes care of everything else. It's a small thing, but man, does it simplify a whole bunch of issues!

[1] Actually, I wouldn't mind updating the CIL book either, but that has less of a chance at a 2nd edition than the attributes book!

[2] That's the fun of being a consultant. You have to sign NDAs, even though what you're developing is pretty much run-of-the-mill. I make every effort to ensure I don't violate them, which is why I make sure I don't release any client-specific code.

> Published: 12.09.2004 11:33:59 AM CST