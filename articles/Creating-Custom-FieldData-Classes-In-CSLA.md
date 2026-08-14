---
title: Creating Custom FieldData Classes in CSLA
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Creating Custom FieldData Classes in CSLA

Recently I've been thrust back into the world of [CSLA](https://cslanet.com/). I worked with it for a while when I wrote my custom blog management tool, but that's the only time I've ever found a need to use the functionality CSLA contains. My current client is doing a lot of business object development and it made sense to have them use CSLA for that layer. I have to admit, when I looked at the new features in CSLA I was really impressed. Writing BOs in the past using CSLA used to be somewhat painful and [Rocky](https://lhotka.net/) has made it much easier with managed properties.

Speaking of properties ... one of the features the client needs to have in the UI is to enable a "save" button only when the user has entered a value that is different from the original value that came from the data source. This is different from the stock behavior in CSLA that considers a BO "dirty" if the property is being set to a value that is different from the one it currently has. Rocky anticipated that people may want this behavior and added extensiblity into CSLA to allow developers to write their own `FieldData<T>` class to handle this feature (check out the "FieldData<T> Class" section in Chapter 7 of the 2008 version of the CSLA book for details). However, as I found out, trying to actually do it was problematic. Fortunately, Rocky and I talked about this and he's changed CSLA (in 3.7.1) to make custom `FieldData<T>` classes relatively straightforward to do. It's still a little trickly, and so I've decided to create this blog post to illustrate how it's done.

Note: this code is based on the current 4.0 beta version of CSLA. This code should work with 3.8 and above. If you're using a previous version of CSLA this code won't work.

To start off, let's say you had a Person BO that looked liked this:

```c#
[Serializable]
public sealed class Person : BusinessCore<Person>
{
  private static PropertyInfo<int> ageProperty =
    Person.RegisterProperty<int>(e => e.Age);
  private static PropertyInfo<string> firstNameProperty =
    Person.RegisterProperty<string>(e => e.FirstName);
  private static PropertyInfo<string> lastNameProperty =
    Person.RegisterProperty<string>(e => e.LastName);

  private Person()
    : base()
  {
  }

  private void DataPortal_Fetch(SingleCriteria<Person, int> criteria)
  {
    if(criteria.Value != 2047)
    {
      throw new NotSupportedException();
    }
        
    using(this.BypassPropertyChecks)
    {
      this.Age = 22;
      this.FirstName = "Joe";
      this.LastName = "Smith";
    }
  }

  protected override void DataPortal_Update()
  {
  }

  public static Person Fetch(int id)
  {
    return DataPortal.Fetch<Person>(new SingleCriteria<Person, int>(id));
  }

  public int Age
  {
    get
    {
      return this.GetProperty(Person.ageProperty);
    }
    set
    {
      this.SetProperty(Person.ageProperty, value);
    }
  }

  public string FirstName
  {
    get
    {
      return this.GetProperty(Person.firstNameProperty);
    }
    set
    {
      this.SetProperty(Person.firstNameProperty, value);
    }
  }

  public string LastName
  {
    get
    {
      return this.GetProperty(Person.lastNameProperty);
    }
    set
    {
      this.SetProperty(Person.lastNameProperty, value);
    }
  }
}
```

This looks like a fair amount of code, but it's just a simple person definition with three properties: `FirstName`, `LastName`, and `Age`. I only have a `Fetch()` method which uses the `DataPortal` to get a `Person` - I didn't include the other usual CRUD suspects to handle creation, updating and deleting to keep the example simple. The only valid `Id` is 2047, and if that's what's given I set the properties to hard-coded values. In a real-world situation, you'd use a data access layer or a repository or some other form of data magic to get the person data, but I'm not really concerned about that in this case. What I **am** concerned about is this:

```c#
[TestMethod]
public void ChangeLastNameUsingSameValue()
{
  var person = Person.Fetch(2047);
  var lastName = person.LastName;
  person.LastName = Guid.NewGuid().ToString("N");
  person.LastName = lastName;
  Assert.IsFalse(person.IsDirty);
}
```

This illustrates the situation where a user gets a person, changes the last name ... but then changes it back to the original value. In this scenario, the user is expecting that the data has not changed so the BO should return `false` for `IsDirty`. The out-of-the-box implementation of CSLA doesn't keep track of the dirty state based on the original value, so this test fails.

So ... how do we support this feature? It's a bit of work, but most of it is boilerplace. The first thing is to create a class that implements `Csla.Core.IPropertyInfoFactory`:

```c#
public sealed class PropertyInformationFactory : IPropertyInfoFactory
{
  public PropertyInfo<T> Create<T>(Type containingType, string name, string friendlyName, 
    T defaultValue, RelationshipTypes relationship)
  {
    return new PropertyInformationUsingOriginalValue<T>(containingType, name, friendlyName, 
      defaultValue, relationship);
  }

  public PropertyInfo<T> Create<T>(Type containingType, string name, string friendlyName, 
    T defaultValue)
  {
    return new PropertyInformationUsingOriginalValue<T>(containingType, name, friendlyName, 
      defaultValue);
  }

  public PropertyInfo<T> Create<T>(Type containingType, string name, string friendlyName, 
    RelationshipTypes relationship)
  {
    return new PropertyInformationUsingOriginalValue<T>(containingType, name, friendlyName, 
      relationship);
  }

  public PropertyInfo<T> Create<T>(Type containingType, string name, string friendlyName)
  {
    return new PropertyInformationUsingOriginalValue<T>(containingType, name, friendlyName);
  }

  public PropertyInfo<T> Create<T>(Type containingType, string name)
  {
    return new PropertyInformationUsingOriginalValue<T>(containingType, name);
  }
}
```

Later on I'll show you how you use this class to inject the custom `FieldData<T>`-based objects I'm making. As you can see, each method returns a `PropertyInformationUsingOriginalValue<T>`-based object - the class looks like this:

```c#
public sealed class PropertyInformationUsingOriginalValue<T>
  : PropertyInfo<T>
{
  public PropertyInformationUsingOriginalValue(Type containingType, string name)
    : base(name)
  {
    this.ContainingType = containingType;
  }

  public PropertyInformationUsingOriginalValue(Type containingType, string name, string friendlyName)
    : base(name, friendlyName)
  {
    this.ContainingType = containingType;
  }

  public PropertyInformationUsingOriginalValue(Type containingType, string name, string friendlyName,
    RelationshipTypes relationship)
    : base(name, friendlyName, relationship)
  {
    this.ContainingType = containingType;
  }

  public PropertyInformationUsingOriginalValue(Type containingType, string name, string friendlyName,
    T defaultValue)
    : base(name, friendlyName, defaultValue)
  {
    this.ContainingType = containingType;
  }

  public PropertyInformationUsingOriginalValue(Type containingType, string name, string friendlyName,
    T defaultValue, RelationshipTypes relationship)
    : base(name, friendlyName, defaultValue, relationship)
  {
    this.ContainingType = containingType;
  }

  protected override IFieldData NewFieldData(string name)
  {
    return typeof(T).IsAssignableFrom(typeof(string)) ?
      new FieldDataUsingOriginalValueViaHashCode<T>(name) as IFieldData :
      new FieldDataUsingOriginalValueViaDuplicate<T>(name) as IFieldData;
  }

  private Type ContainingType { get; set; }
}
```

The key here is `NewFieldData()`. There's two ways I'm handling duplicate information. If the field type is a `string`, then I don't want to hold on to the exact original value as that could get expensive from a memory standpoint. Instead, I'm going to hold on to the hash value of the `string` via the `FieldDataUsingOriginalValueViaHashCode<T>` class. This isn't 100% perfect, but for most cases this will be able to indentify when the current value is different from the original one [1]. For all other types, I'll store the original value via `FieldDataUsingOriginalValueViaDuplicate<T>`. Of course, your needs may be different from what I'm doing in this example - the point to take away here is that this is the place where you'll want to generate your custom `FieldData<T>` class based on property-related information.

Both of these classes inherit from `FieldDataUsingOriginalValue<T, TOriginal>`, so let's look at that class first:

```c#
[Serializable]
public abstract class FieldDataUsingOriginalValue<T, TOriginal>
  : FieldData<T>
{
  protected FieldDataUsingOriginalValue(string name)
    : base(name) { }

  protected abstract bool HasValueChanged();

  public override void MarkClean()
  {
    base.MarkClean();
    this.SetOriginalValue(this.Value);
  }
            
  protected abstract void SetOriginalValue(T value);

  private bool HasBeenSet { get; set; }

  public override bool IsDirty
  {
    get
    {
      return !this.HasBeenSet ? base.IsDirty : this.HasValueChanged();
    }
  }

  protected TOriginal OriginalValue { get; set; }

  public override T Value
  {
    get
    {
      return base.Value;
    }
    set
    {
      if(!this.HasBeenSet)
      {
        this.SetOriginalValue(value);
        this.HasBeenSet = true;
      }
                
      base.Value = value;
    }
  }
}
```

The first time `Value` is called, I capture that original value so I can use it for comparison purposes. This original value will be stored by the subclass via `SetOriginalValue()` in whatever format they need - i.e. hash code, duplicate, etc. By stuffing this call tracking into a base class, the two subclasses don't have to concern themselves with it; they can focus on how they keep track of changed state. I also reset the original value when `MarkClean()` is called - this is needed when the object is saved as the original value needs to be reset. Let's look at how `FieldDataUsingOriginalValueViaDuplicate<T>` does it:

```c#
[Serializable]
public sealed class FieldDataUsingOriginalValueViaDuplicate<T>
  : FieldDataUsingOriginalValue<T, T>
{
  public FieldDataUsingOriginalValueViaDuplicate(string name)
    : base(name) { }

  protected override bool HasValueChanged()
  {
    var hasValueChanged = false;
            
    if(typeof(T).IsValueType)
    {
      hasValueChanged = !this.OriginalValue.Equals(this.Value);
    }
    else
    {
      var originalIsNotNull = this.OriginalValue != null;
      var currentIsNotNull = this.Value != null;

      if(originalIsNotNull != currentIsNotNull)
      {
        hasValueChanged = true;
      }
      else
      {
        if(originalIsNotNull)
        {
          var child = this.Value as ITrackStatus;

          if(child != null)
          {
            hasValueChanged = child.IsDirty;
          }
          else
          {
            hasValueChanged = !this.OriginalValue.Equals(this.Value);
          }
        }
      }
    }
            
    return hasValueChanged;
  }

  protected override void SetOriginalValue(T value)
  {
    this.OriginalValue = value;
  }
}
```

As you can see, `HasValueChanged()` has the "smarts" in it to determine if the value has really changed. It's not overly complicated, but I have to keep track of reference vs. value type and child objects (`ITrackStatus`). Let's compare this implementation with `FieldDataUsingOriginalValueViaHashCode<T>`:

```c#
[Serializable]
public sealed class FieldDataUsingOriginalValueViaHashCode<T>
  : FieldDataUsingOriginalValue<T, int>
{
  public FieldDataUsingOriginalValueViaHashCode(string name)
    : base(name) { }

  protected override bool HasValueChanged()
  {
    var hasValueChanged = false;

    if(typeof(T).IsValueType)
    {
      hasValueChanged = !(this.OriginalValue == this.Value.GetHashCode());
    }
    else
    {
      var child = this.Value as ITrackStatus;

      if(child != null)
      {
        hasValueChanged = child.IsDirty;
      }
      else
      {
        hasValueChanged = this.Value != null ?
          this.OriginalValue != this.Value.GetHashCode() :
          this.OriginalValue != 0;
      }
    }

    return hasValueChanged;
  }

  protected override void SetOriginalValue(T value)
  {
    this.OriginalValue = (value != null) ? value.GetHashCode() : 0;
  }
}
```

The only key difference is that I'm capturing the hash code value in `SetOriginalValue()` to use for comparison purposes.

I also have a `BusinessCore<T>` class that I use for my business objects. It's recommended that you create your own derivations of the core CSLA stereotypes so any changes you make to BO behavior is easily reflected across all of your classes - here's what mine looks like:

```c#
[Serializable]
public abstract class BusinessCore<T>
  : BusinessBase<T>
  where T : BusinessCore<T>
{
  protected BusinessCore()
    : base() { }

  protected override void Child_OnDataPortalInvokeComplete(DataPortalEventArgs e)
  {
    base.Child_OnDataPortalInvokeComplete(e);
    this.HandleOnDataPortalComplete(e);
  }

  protected override void DataPortal_OnDataPortalInvokeComplete(DataPortalEventArgs e)
  {
    base.DataPortal_OnDataPortalInvokeComplete(e);
    this.HandleOnDataPortalComplete(e);
  }

  private void HandleOnDataPortalComplete(DataPortalEventArgs e)
  {
    if(e.Operation == DataPortalOperations.Create)
    {
      this.MarkClean();
    }
  }

  public override bool IsSelfDirty
  {
    get
    {
      var isSelfDirty = false;

      foreach(var registeredProperty in this.FieldManager.GetRegisteredProperties())
      {
        if(registeredProperty.RelationshipType.HasFlag(RelationshipTypes.Child) &&
          this.FieldManager.IsFieldDirty(registeredProperty))
        {
          isSelfDirty = true;
          break;
        }
      }

      return isSelfDirty;
    }
  }
}
```

The key override is `IsSelfDirty`. CSLA does some "caching" with the dirty status in that, once an object goes dirty, it (more or less) assumes it'll stay dirty. To change that behavior, I had to override these properties to check the status every time. This isn't as performant as CSLA's default implementation  but it's necessary if you want the dirty status to reflect changed status based on the original value. I also hook the `DataPortal` complete events to mark a new object "clean". This is a deviation from standard CSLA behavior and frankly isn't necessary for the field management discussion, but I usually do it as it makes more sense to me for a new object to be in the "clean" state. If you don't want this, simply remove those methods.

The final piece is in the test class:

```c#
[AssemblyInitialize]
public static void AssemblyInitialize(TestContext context)
{
  PropertyInfoFactory.Factory = new PropertyInformationFactory();
}
```

The point to take away here is you must set the `Factory` property somewhere at the "beginning" of your host's initialization. That will differ from host to host (WCF, WPF, WinService, etc.) - for the purposes of this example setting it when the test class initializes is sufficient. Keep in mind that doing this in every test won't work as expected because managed properties are resolved in the class's static constructor, so once that's done for a given class, it will be using whatever factory was set at that time. Changing the factory after that point is meaningless.

With all of this in place, our test will finally pass!

I realize this seems like a lot of work, but really it comes down to the following steps:

* Create a custom property factory
* Create the custom `FieldData<T>` classes you need and return them in `NewFieldData()`
* Create a custom `BusinessBase<T>` class (or whatever base CSLA class you're using) that manages `IsDirty` and `IsSelfDirty`
* Set the `Factory` property at the beginning of the application's execution

Enjoy!

[1] You could also use System.Security.Cryptography.HashAlgorithm-based objects to reduce the collisions even more.

> Published: 09.16.2009 10:30:06 PM CST