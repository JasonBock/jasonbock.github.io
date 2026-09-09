---
title: Serializing Behavior-Based Non-Serializable Fields in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Serializing Behavior-Based Non-Serializable Fields in .NET

Let's say you have the following C# class defined:

```c#
using System;
using System.IO;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters;
using System.Runtime.Serialization.Formatters.Binary;

public class ObjectWrapper
{
  private object m_inner = null;
  private IFormatter m_formatter = null;
    
  public ObjectWrapper(Object ObjectToWrap, IFormatter SerializationStrategy)
  {
    int hasSerializableAttrib = (int)ObjectToWrap.GetType().Attributes & 
      (int)TypeAttributes.Serializable;
        
    if(0 == hasSerializableAttrib)
    {
      throw new ApplicationException("The given object is not serializable.");
    }

    this.m_inner = ObjectToWrap;
    this.m_formatter = SerializationStrategy;
  }

  public object InnerObject
  {
    get
    {
      MemoryStream ms = new MemoryStream();
      ms.Write(this.m_inner, 0, (int)this.m_inner.Length);
      ms.Position = 0;
      return(this.m_formatter.Deserialize(ms));
    }
        
    set
    {
      MemoryStream ms = new MemoryStream();
      this.m_formatter.Serialize(ms, value);
      this.m_inner = ms.GetBuffer();
      ms.Close();
    }
  }
}
```

It's a simplistic class in that all it does is serialize and deserialize a given object. (This actually has a purpose - I'll explain in a moment). However, let's say that you want to serialize `ObjectWrapper`:

```c#
[Serializable]
public class ObjectWrapper
{
  private object m_inner = null;
  private IFormatter m_formatter = null;
  //...
}
```

That's the easy way to do it, but `IFormatter` is not serializable. OK ...:

```c#
[Serializable]
public class ObjectWrapper
{
  private object m_inner = null;
  [NonSerialized]
  private IFormatter m_formatter = null;
//...
}
```

Now you won't get an exception when you serialize, but what if you store the `ObjectWrapper` instance to disk? When you rehydrate it, `m_formatter` will be `null`, so any `InnerObject` call will fail miserably. Sure, you could add a check whenever the reference is used, but that can get messy.

The easiest way to solve this is to handle the serialization yourself:

```c#
[Serializable]
public class ObjectWrapper : ISerializable
{
  private byte[] m_inner = null;
  private IFormatter m_formatter = null;

  public ObjectWrapper(object ObjectToWrap, IFormatter SerializationStrategy)
  {
    this.m_formatter = SerializationStrategy;
    MemoryStream ms = new MemoryStream();
    this.m_formatter.Serialize(ms, ObjectToWrap);
    this.m_inner = ms.GetBuffer();
    ms.Close();
  }
    
  private ObjectWrapper(SerializationInfo si, StreamingContext stc)
  {
    int hasSerializableAttrib = (int)ObjectToWrap.GetType().Attributes & 
      (int)TypeAttributes.Serializable;
		
    if(0 == hasSerializableAttrib)
    {
      throw new ApplicationException("The given object is not serializable.");
    }

    String formatterType = (String)si.GetValue("FormatterStrategy", 
      typeof(System.String));
    this.m_formatter = (IFormatter)Activator.CreateInstance(Type.GetType(formatterType));
    this.m_inner = (byte[])si.GetValue("Inner", typeof(System.Byte[]));
  }

  public object InnerObject
  {
    //  See previous code snippet...
  }

  public void GetObjectData(SerializationInfo si, StreamingContext scx)
  {
    si.AddValue("FormatterStrategy", this.m_formatter.GetType().FullName);
    si.AddValue("Inner", this.m_inner);  
  }
}
```

When `ObjectWrapper` is serialized, the type name of `m_formatter` is stored. When it's rehydrated, we use `CreateInstance()` to recreate the strategy.

This works very well if you have an object that you need to serialize its' behavioral aspect, especially if it isn't serializable in the first place. We don't need to actually store the formatting type; we only need enough information to recreate it when necessary.

I just ran into this particular scenario when I was writing a section in a chapter for a book I'm writing on .NET. I wanted to write a class that would allow a user to encrypt an object using someone's public key, thereby sealing its state until the owner of the private key opened it. (If you're a Java developer, this will sound familiar). Therefore, I needed to serialize the given object, but I didn't want the recipient of the sealed object to know how the sender serialized the object - I wanted to persist the implementation that `IFormatter` binded to. The subsequent path of problems I ran into is what we just went over.

> Published: 02.28.2002 12:00:00 AM CST