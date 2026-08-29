---
title: The Pain of NUnitAsp, or, Parsing HTML ... And "Random" Data
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Pain of NUnitAsp, or, Parsing HTML ... And "Random" Data

Ah, the thrill of being on a project again. I love spending hours on something that should've been easier, but just seemed to throw me for a loop and a half. Basically, I wanted to parse the response from a page submission in an NUnitAsp test, but seeing that it was HTML the `XmlDocument` didn't like it. After some digging, I saw that there was a `SgmlReader` class, which didn't surprise me as NUnitAsp must be doing some SGML parsing to support all the linking they do with page elements and test elements and whatnot. The problem is, I couldn't get `SgmlReader` to parse my HTML - something about a null reference in a lazy loading of the DTD. So, I popped open Reflector to see where NUnitAsp's code was doing some SGML parsing, and, basically, this is what I came up with:

```c#
private XmlDocument GetDocumentFromPageData(string pageData)
{
  string dtdResource = "NUnit.Extensions.Asp.Sgml.Html.dtd";
  Stream dtdStream = 
    typeof(SgmlDtd).Assembly.GetManifestResourceStream(dtdResource);
  StreamReader dtdReader = new StreamReader(dtdStream);

  SgmlReader reader = new SgmlReader();
  reader.DocType = "HTML";
  reader.Dtd = SgmlDtd.Parse((Uri)null, "HTML", (string)null, 
    dtdReader, (string)null, (string)null, reader.NameTable);
  reader.InputStream = new StreamReader(
    new MemoryStream(ASCIIEncoding.Default.GetBytes(pageData)));

  XmlDocument pageDocument = new XmlDocument();
  pageDocument.XmlResolver = null;
  pageDocument.Load(reader);
    
  return pageDocument;
}
```

Don't ask. All I know is that it works in converting the `Browser.CurrentPageText` into an `XmlDocument`. YMMV ...

I also found out that generating random data may not seem so random:

```c#
private string GenerateRandomString(int wordSize, bool isUpperCase)
{
  StringBuilder result = new StringBuilder();
  Random rnd = new Random(Environment.TickCount);

  for(int i = 0; i < wordSize; i++)
  {
    if(isUpperCase == true)
    {
      result.Append((char)rnd.Next(65, 91));
    }
    else
    {
      result.Append((char)rnd.Next(97, 123));
    }
  }

  return result.ToString();
}

// ...

string password1 = this.GenerateRandomString(12, false);
string password2 = this.GenerateRandomString(12, false);
```

Much to my surprise, my test to ensure the page reported an error if the values in two password boxes failed worked. In other words, the password strings were the same. This fixed it:

```c#
string password1 = this.GenerateRandomString(12, false);
System.Threading.Thread.Sleep(3);
string password2 = this.GenerateRandomString(12, false);
```

This results in the tick count being different such that the strings were different. I'm sure a sleep value of 1 would work, but I liked the number 3 today for some reason. Who said .NET code was slow?

> Published: 05.04.2005 03:16:07 PM CST