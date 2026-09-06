---
title: Read-Only Configurable Properties and Unit Testing
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Read-Only Configurable Properties and Unit Testing

As the holiday season (and the subsequent travel) draws near, I'll be away from my blog for about a week. Just so I don't forget, Happy Holidays :).

Anyway, I ran into an interesting quirk yesterday during a unit test session. It reared its head primarily because unit tests aren't very prevalent at the current client and therefore tests aren't always run. This is a bad thing, but ... well, I'm trying to make sure that whatever I add to the system has tests and that I run them before a check-in. Unfortunately, manual processes are easy to forget to do, and I had changed some code without running the tests, and when I added a new class I decided to run the tests, and ... kablamoo!! It wasn't a huge deal as everything was working as expected (as you're about see), but, still, I gotta get the client into more of a agile mindset.

All right, here's the situation, distilled to the core. I have a class called `MyClass` (I'm not creative today, but I refuse to use `Quux` and `Foo`, so that's the best name I could come up with), with a property called `MyProperty`:

```c#
public class MyClass : UserControl
{
  protected bool myProperty = false;

  public MyClass() : base()
  {
    this.myProperty = HelperClass.GetPropertyFromConfiguration();
  }

  public bool MyProperty
  {
    get
    {
      return this.myProperty;
    }
    set
    {
      if(this.DesignMode == true)
      {
        this.myProperty = value;
      }
    }
  }
}
```

Basically, the value of `MyProperty` can only be set when the control is being hosted in a Designer. At runtime, the property is only set during construction and cannot be changed. The field's value is read from the .config file, which is what `GetPropertyFromConfiguration()`. Originally, the code didn't work this way, but after a while there was a reason to make the property value read-only at runtime. Yes, there's a reason that I have my code working this way, but it would take too much time to get into the reasons and rationale, so for the purposes of this discussion please just accept that what's being done in the code is good.

The problem shows up in a unit test:

```c#
[Test()]
public void CheckProperty()
{
  MyClass customControl = new MyClass();
  Assert.IsFalse(customControl.MyProperty,
    "The property value (after construction) is incorrect.");

  customControl.MyProperty = true;
  Assert.IsTrue(customControl.MyProperty,
    "The property value (when MyProperty is set to true) is incorrect.");
}
```

This unit test was written before `MyProperty` became read-only at runtime, so it worked without a hitch. Unfortunately, because the tests are not being run in a continuous fashion with the build, I didn't notice the failures until yesterday. Oops!

I came up with a simple workaround. I added another configuration value called `CanChangeProperty`, and I adjusted the property's setter as follows:

```c#
set
{
  if(this.DesignMode == true ||
    HelperClass.GetCanChangeProperty() == true)
  {
    this.myProperty = value;
  }
}
```

Then I added a .dll.config file to my unit test project so TestDriven.NET could pick it up, and I set `CanChangeProperty` to true. After the change, my tests ran fine.

Now, this is a little "risky" in that I can make the property writeable at runtime by flipping a simple switch in the configuration file. But for now the solution will suffice. I'm wondering if a better approach would be to somehow "look" at the environment that the unit test is running in, and if it's being run by TestDriven.NET or NUnit then it's OK to make the property writeable. That may be a bit more resilient to accidental configuration changes.

> Published: 12.22.2004 12:40:48 PM CST