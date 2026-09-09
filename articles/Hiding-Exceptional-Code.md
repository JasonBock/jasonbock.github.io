---
title: Hiding Exceptional Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Hiding Exceptional Code

## What is Wrong With This Code?

Language: Java 

```java
protected static Document createDocument(String pathName)
  throws IOException
{
  Document document = null;
    
  if (pathName != null && !pathName.equals(""))
  {
    document = (Document)DocumentCacheImpl.get( pathName );
        
    if(document == null)
    {
      document = XmlReader.getDOM(pathName, true);
      DocumentCacheImpl.put( pathName, document );
    }
    
    synchronized(document)
    {
      document = (Document) document.cloneNode(true);
    }
  }
  return document;
}
```

The purpose of this method is to create an XML document; More specifically, a clone of a document. The argument `pathName` is a full file path to a given XML file. 

The problem wasn’t obvious when I first started to run into issues with this method. Occasionally I'd see `NullPointerException`s occurring when this code was run. I first thought it was due to the document reference being equal to `null`, which would cause the `synchronized` statement to fail. However, I eventually realized that `put()` would fail if document was `null` (you'll see why in a second), so the issue lies one line of code away. 

The problem here is `getDOM()`. The singleton `DocumentCacheImpl` is used to cache XML documents in memory – presumably for performance reasons (although I’m not sure if it helped, but that’s a different story). If it doesn't exist in memory, `XmlReader` loads the DOM, and document is then stored in the cache. `getDOM()` looks like this:

```java
public static synchronized Document getDOM(String fileName, 
  boolean validate)
{
  try
  {
    return getParser().parse(fileName,validate);
  }
  catch (Exception e)
  {
    e.printStackTrace();
    return null;
  }
}
```

To give an example on exception handling from another language, Eiffel puts the burden of handling exceptions squarely on the shoulders of the method that caused it - it cannot shove it under the covers. However, as `getDOM()` demonstrates, hiding exceptions in Java is pretty easy. If an invalid path is entered, `parse()` will throw an exception, but the exception is lost, and we get a null reference back. 

Now we store this null reference in `DocumentCacheImpl`, which is just a wrapper around a hashtable. `put()` does not accept a null reference for either the key or the value, so a `NullPointerException` is raised. 

## Possible Solutions

So how can we fix this? You could simply check the return value from `getDOM()` for a `null` value like this: 

```java
if(document == null)
{
  document = XmlReader.getDOM(pathName, true);
  if(document != null)
  {
    DocumentCacheImpl.put( pathName, document );
  }
}
```

However, I think there’s a better solution. First, I would change the arguments to both `createDocument` and `getDOM` to take a `File` type. The methods could easily check to see if the file exists by calling `exists()` on the `File` object (I'd also make sure the given arguments are not `null`). Next, I would not hide the exception in `getDOM()`. Finally, I would change `createDocument()` to re-check the document reference before it makes the `put()` call on the hashtable. 

Here's what the changes look like: 

```java
public static synchronized Document getDOM(File xmlFile, 
  boolean validate) throws Exception
{
  if(xmlFile != null && xmlFile.exists())
  {  
    return getParser().parse(xmlFile.getName(), validate);
  }
  else
  {
    throw new Exception("The given XML file does not exist.");
  }
}

protected static Document createDocument(File xmlFile) throws Exception
{
  Document document = null;
    
  if(xmlFile != null && xmlFile.exists())
  {
    document = (Document)DocumentCacheImpl.get(xmlFile.getName());
        
    if(document == null)
    {
      document = XmlReader.getDOM(xmlFile, true);
      DocumentCacheImpl.put(xmlFile.getName(), document);
    }
        
    synchronized(document)
    {
      document = (Document)document.cloneNode(true);
    }
  }
  else
  {
    throw new Exception("The given XML file does not exist.");
  }
  return document;
}
```

Note that we don't have to check for a `null` document on the return value from `getDOM()` as it will either return a valid reference or an exception will be thrown. I've also made the thrown exception in `createDocument()` to be of an `Exception` type and not an `IOException`. 

To be honest, I can’t remember the exact reason why I found the error in the first place. I believe that I was passing in an invalid path, and this was causing the program to bomb. One could argue that it was my fault for passing in an invalid path in the first place, and I would agree with that assessment. However, the method needed to handle such exceptional conditions. The method simply assumed that the given path was correct, and there’s no way to enforce this other than by convention. But if the argument was strongly typed to be a `File`-based object, such conditions can be tested rather easily, which cuts down on time spent debugging. 

> Note: The argument’s name changes slightly in the two methods. In `createDocument()`, it’s `pathName`, but in `getDOM()`, it’s `fileName`. To me, that means two different things – a file path for the first argument, but a file name for the second. Others may interpret this differently, but by using a `File` object the ambiguity is removed.  

## Summary 

1. Don’t hide exceptions. If the client did something that you didn’t expect, report it, don’t bury it.
2. Type your variables correctly. Using a `String` for a file path works, but using a `File` object is better.

> Published: 04.06.2001