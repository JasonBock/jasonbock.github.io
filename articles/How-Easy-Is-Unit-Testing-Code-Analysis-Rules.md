---
title: How Easy Is Unit Testing Code Analysis Rules?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# How Easy Is Unit Testing Code Analysis Rules?

In my [last post](https://jasonbock.net/articles/Using-Custom-CodeAnalysis-Rules-In-Your-CSLA-Projects), I mentioned that, in the past, I've found unit testing custom Code Analysis rules to be a pain ([here's](https://jasonbock.net/articles/Writing-Custom-Rules-For-VS-2008-Code-Analysis-What-A-Pain) an example of what I'm talking about [1]). I'm not the only one to have run into issues in the past either. However, now it's just freaking easy. Here's a quick walkthrough of one of the CSLA rules I wrote and how I test it.

In CSLA, your business objects should be marked as `[Serializable]`. This is pretty easy to do ... and pretty easy to forget. So I wrote a rule called `ClassesMustBeMarkedWithSerializableAttributeRule` in my custom rules to check for that. Here's how it works:

```c#
using Csla;
using Csla.Core;
using Microsoft.FxCop.Sdk;
using System;
using System.Collections.Generic;

namespace Csla.Rules
{
  public sealed class ClassesMustBeMarkedWithSerializableAttributeRule
    : BaseRule
  {
    public ClassesMustBeMarkedWithSerializableAttributeRule() :
      base(typeof(ClassesMustBeMarkedWithSerializableAttributeRule).Name)
    {
    }

    public override ProblemCollection Check(TypeNode type)
    {
      if (type == null)
      {
        throw new ArgumentNullException("type");
      }

      if(Utilities.IsStereotype(type))
      {
        if((type.Flags & TypeFlags.Serializable) == TypeFlags.None)
        {
          this.Problems.Add(new Problem(this.GetResolution(type), type));
        }
      }

      return this.Problems;
    }
  }
}
```

`Utilities.IsStereotype()` checks for two things: is the type assignable from `IBusinessObject` or `EditableRootListBase<>`? If either one is true, then it checks to see if the type is serializable via the `Flags` property. `SerializableAttribute` is one of those funky attributes that won't show up in a `GetAttribute()` call; essentially it gets shoved down into a bit flag in your metadata, so you have to determine if it's there through other means.

As you can see, the rule is pretty straightforward. Now, moving on to the tests - I'll show you one of them:

```c#
using Csla.Rules.Extensions;
using Csla.Rules.Tests.Types;
using Microsoft.FxCop.Sdk;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace Csla.Rules.Tests
{
  [TestClass]
  public sealed class ClassesMustBeMarkedWithSerializableAttributeRuleTests
  {
    [TestMethod]
    public void CheckClassThatImplementsIBusinessObjectWithAttribute()
    {
      TestHelpers.CheckRule(
        new ClassesMustBeMarkedWithSerializableAttributeRule(),
        typeof(BusinessBaseWithSerializableAttribute), 0);
    }

    // More tests go here ...
  }
}
```

I have a bunch of classes defines that either satisfy or fail the rule in my test project, like `BusinessBaseWithSerializableAttribute`. Here's what that looks like:

```c#
using Csla;
using System;

namespace Csla.Rules.Tests.Types
{
  [Serializable]
  internal sealed class BusinessBaseWithSerializableAttribute
    : BusinessBase<BusinessBaseWithSerializableAttribute>
  {
  }
}
```

Depending on how your custom Code Analysis rule works, you may have to implement other strategies to pass in the right `Member` (e.g. `TypeNode`) into the rule. In this case, all I need is a class that inherits from `BusinessBase<>` (which implements `IBusinessObject`) and has `[Serializable]` on it.

My `CheckRule()` method does all the work ... which isn't a lot:

```c#
using Csla.Rules;
using Csla.Rules.Extensions;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace Csla.Rules.Tests
{
  internal static class TestHelpers
  {
    internal static void CheckRule(BaseRule rule, Type type, int expectedProblemCount)
    {
      var targetAssembly = type.GetAssembly();
      var targetType = targetAssembly.GetType(type);
      var problems = rule.Check(targetType);
      Assert.AreEqual(expectedProblemCount, problems.Count);
    }
  }
}
```

I have a couple of extension methods to make the transition from the `System.Reflection` world to the `Microsoft.FxCop.Sdk` world, like `GetAssembly()` and `GetType()`. Once I have those transitional values in place, I call my rule, and check the `ProblemCollection` return value.

That's it. It's really not that much work, and I now know if my rules work or not without having to host them in a VS project. I still do that from time to time just to make sure it does work in the "real world" scenario, but that's not the default test mode. I just write unit tests now, and developing custom Code Analysis rules is now a lot easier and more appealing that it used to be.

[1] In retrospect, using `RuleUtilities.GetAssembly()` might have been a mistake. It still returns `null` in 2010, but `AssemblyNode.GetAssembly()` works just fine.

> Published: 04.20.2010 08:42:03 AM CST