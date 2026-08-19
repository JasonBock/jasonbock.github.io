---
title: A General, Dynamic ToString() Implementation
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A General, Dynamic ToString() Implementation

Recently I saw a post where someone came up with a way via Reflection to produce a string that would be returned by `ToString()`. This would contain the values of the properties on the current object. I liked the idea, but in my comment I listed a number of things I didn't like about it. I also mentioned that this is something that `DynamicMethod` may be a better choice for, but that's not trivial to do. So I decided to show how it's done (because, while it's not hard to do, it's not trivial either).

First, let's define an interface with a base class implementation:

```c#
public interface ICustomer
{
  int Age
  {
    get;
    set;
  }

  Guid Id
  {
    get;
    set;
  }
    
  string FirstName
  {
    get;
    set;
  }

  string LastName
  {
    get;
    set;
  }
}

public abstract class Customer : ICustomer
{
  protected Customer()
    : base()
  {
    this.Id = Guid.NewGuid();
  }

  public int Age
  {
    get;
    set;
  }

  public Guid Id
  {
    get;
    set;
  }

  public string FirstName
  {
    get;
    set;
  }

  public string LastName
  {
    get;
    set;
  }
}
```

Here's the Reflection idea:

```c#
internal sealed class CustomerReflection : Customer
{
  public override string ToString()
  {
    return ToStringReflection.RunToString(this);
  }
}

public static class ToStringReflection
{
  public static string RunToString(object target)
  {
    return string.Join(" || ", Array.ConvertAll<PropertyInfo, string>(
      target.GetType().GetProperties(BindingFlags.Instance | BindingFlags.Public),
      prop => prop.CanRead ? string.Format("{0} : {1}", 
        prop.Name, prop.GetValue(target, null)) : string.Empty));        
  }
}
```

I changed the code around a bit, but basically it's the same idea from the post. All of the public instance properties that have a getter are added to the result.

Now, before I dive into how I did this with a `DynamicMethod`, consider how you'd do this all by yourself:

```c#
public sealed class CustomerHardCoded : Customer
{
  private const string Separator = " || ";
    
  public override string ToString()
  {
    var toString = new StringBuilder();

    toString.Append("Age: ").Append(this.Age.ToString());
    toString.Append(CustomerHardCoded.Separator);
    toString.Append("Id: ").Append(this.Id.ToString());
    toString.Append(CustomerHardCoded.Separator);
    toString.Append("LastName: ").Append(this.LastName);
    toString.Append(CustomerHardCoded.Separator);
    toString.Append("FirstName: ").Append(this.FirstName);
        
    return toString.ToString();
  }
}
```

The reason I show this approach is that I can use the underlying IL to figure out how I need to create the dynamic method, which looks like this:

```c#
public sealed class CustomerDynamicMethod : Customer
{
  public override string ToString()
  {
    return ToStringDynamicMethod.RunToString(this);
  }
}

internal delegate string ToStringDelegate<T>(T target);

public static class ToStringDynamicMethod
{
  private const string Separator = " || ";
    
  private static Dictionary<Type, Delegate> toStrings =
    new Dictionary<Type, Delegate>();
        
  public static string RunToString<T>(T target)
  {
    ToStringDelegate<T> toString = null;

    Type targetType = target.GetType();

    if(ToStringDynamicMethod.toStrings.ContainsKey(targetType))
    {
      toString = ToStringDynamicMethod.toStrings[targetType] as ToStringDelegate<T>;
    }
    else
    {
      toString = ToStringDynamicMethod.CreateToString(target);
      ToStringDynamicMethod.toStrings.Add(targetType, toString);
    }

    return toString(target);
  }
    
  private static ToStringDelegate<T> CreateToString<T>(T target)
  {
    Type type = typeof(T);
        
    DynamicMethod toString = new DynamicMethod("ToString" + typeof(T).GetHashCode().ToString(), 
      typeof(string), new Type[] { typeof(T) }, typeof(ToStringDynamicMethod).Module);
        
    ILGenerator generator = toString.GetILGenerator();
        
    PropertyInfo[] properties = type.GetProperties(BindingFlags.Public | BindingFlags.Instance);

    if(properties.Length > 0)
    {
      Type stringBuilderType = typeof(StringBuilder);
            
      generator.Emit(OpCodes.Newobj, stringBuilderType.GetConstructor(Type.EmptyTypes));
      MethodInfo appendMethod = stringBuilderType.GetMethod(
        "Append", new Type[] { typeof(string) });
      MethodInfo toStringMethod = typeof(object).GetMethod("ToString");
            
      for(int i = 0; i < properties.Length; i++)
      {
        PropertyInfo property = properties[i];
                
        if(property.CanRead)
        {
          generator.Emit(OpCodes.Ldstr, property.Name + ": ");
          generator.EmitCall(OpCodes.Callvirt, appendMethod, null);
          generator.Emit(OpCodes.Ldarg_0);

          MethodInfo propertyGet = property.GetGetMethod();

          generator.EmitCall(propertyGet.IsVirtual ? OpCodes.Callvirt : OpCodes.Call,
            propertyGet, null);

          Type returnType = propertyGet.ReturnType;
          if(returnType != typeof(string))
          {
            if(returnType.IsValueType)
            {
              LocalBuilder localReturnType = generator.DeclareLocal(returnType);
              generator.Emit(OpCodes.Stloc, localReturnType);
              generator.Emit(OpCodes.Ldloca, localReturnType);
            }
                        
            MethodInfo returnToStringMethod = returnType.GetMethod("ToString", Type.EmptyTypes);
            generator.EmitCall(OpCodes.Callvirt, returnToStringMethod ?? toStringMethod, null);
          }

          generator.EmitCall(OpCodes.Callvirt, appendMethod, null);

          if(i < properties.Length - 1)
          {
            generator.Emit(OpCodes.Ldstr, ToStringDynamicMethod.Separator);
            generator.EmitCall(OpCodes.Callvirt, appendMethod, null);
          }                    
        }
      }

      generator.EmitCall(OpCodes.Callvirt, toStringMethod, null);
    }
    else
    {
      generator.Emit(OpCodes.Ldstr, string.Empty);
    }            
        
    generator.Emit(OpCodes.Ret);
    
    return (ToStringDelegate<T>)toString.CreateDelegate(typeof(ToStringDelegate<T>));
  }
}
```

Now, how fast is this? Trying each approach the same amount of times (500000) gives me the following (typical) results:

```
Type: CustomerHardCoded, Time: 00:00:01.7237808
Type: CustomerReflection, Time: 00:00:07.5133585
Type: CustomerDynamicMethod, Time: 00:00:01.8811192
```

So the dynamic approach is a little slower than the hard-coded approach, but it's much faster than the Reflection approach. This is a little unfair because I cache the `DynamicMethod` once I've created it. If I didn't do this, then the `DynamicMethod` approach really sucks (in fact I got too impatient to wait for the timing results to come in!).

I've found that when one uses Reflection that does a similar thing against numerous types, doing it as a `DynamicMethod` may be a better approach. It's harder to work with, but the payoff is worth it.

By the way, I used the `ILVisualizer` to help out this time around. It's very nice to see what the `DynamicMethod` looks like during execution, although being able to debug those methods would be nice as well (yes, there is one way to do it but it's not seamless).

> Published: 01.10.2008 12:56:24 PM CST