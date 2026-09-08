---
title: No Description From Attribute
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# No Description From Attribute

OK, I'm confused as hell. Why doesn't this work?

```c#
using System.ServiceProcess;

ServiceProcessDescriptionAttribute spda = new ServiceProcessDescriptionAttribute("SPDA Description");
Console.WriteLine(spda.Description);
```

`Description` doesn't print anything back. Now, `ServiceProcessDescriptionAttribute` descends from `DescriptionAttribute`, and, amazingly, this works:

```c#
using System.ComponentModel;

DescriptionAttribute da = new DescriptionAttribute("DA Description");
Console.WriteLine(da.Description);
```

I've looked at the CIL, and it doesn't help. The `description` field in `DescriptionAttribute` **should** be equal to "SPDA Description" after the constructor is complete. I can understand that some of the code in `get_Description()` may cause the value not be returned, but **right after** the constructor is done, `description` should be OK. The reason I'm diving down into the base class is that I was looking into a way to extract the field's value via reflection, but then I noticed in the debugger that `description` was `null` right after the constructor completed. The only thing that seems weird is that the base constructor is called after the `replaced` field is set, but I'm not sure if that's the reason the initialization isn't working.

Note that this has nothing to do with actually setting the description value for a service. I know this attribute doesn't help and I've already make the P/Invokes to get around it. But I wanted to use this attribute as metadata that I could pull out and use when I make the P/Invokes. I know I could simply use `DescriptionAttribute` but I'd really like to figure out this constuctor weirdness.

I hate to admit it, but I'm stumped. If anybody can give me a correct description of why this doesn't work before I figure it out myself, I'll send you an autographed copy of my CIL book!

> Published: 07.19.2002 03:06:00 PM CST