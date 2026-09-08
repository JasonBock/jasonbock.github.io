---
title: Getting Familiar With NUnit and NAnt
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Getting Familiar With NUnit and NAnt

I forgot to mention yesterday that I spent some time getting familiar with NUnit and NAnt. I've used JUnit for some Java code in the past, but I really like the way the tool has been adopted to take advantage of attributes in .NET. It's no longer necessary to have your methods start with "Test"; just add the `[Test]` attribute and that's it (more or less). NAnt is cool - I can now compile my code and my unit test assembly, run the unit tests, verify the assembly with PEVerify, and do a FxCop evaluation of my assembly all in one shot. I've noticed that the `<nunit>` target doesn't seem to work, so I just used the `<exec>` target on nunit-console.exe, which works just fine. Maybe I don't have something set up in the `<nunit>` correctly, though.

By the way, I've been thinking about the role that unit testing plays if you have design-by-contract (DBC). Now, before anyone who values unit testing jumps down my throat, let me state that I like unit testing. Testing is a very good thing - I'm not knocking it, and I need to get in the habit of doing it more often. But here's my question. Let's say I have a `Book` class that has three properties: `Title`, `Author`, and `Pages`:

```c#
public class Book
{
  private string mTitle = String.Empty;
  private string mAuthor = String.Empty;
  private int mPages = 0;

  private Book() : base() {}

  public Book(string Title, string Author, int Pages)
  {
    this.mTitle = Title;
    this.mAuthor = Author;
    this.mPages = Pages;
  }
}
```

There are three read-only properties for each field so a client can get book information, but I won't show them here. Now, let's say there are some rules a `Book` object needs to follow. Both the title and the author need to be set (i.e. not null and `Length` greater than 0), and neither can be greater than 200 characters in length. The page count must be greater than 0 and less than 10000.

Here's a NUnit test method to see if our object does this for the page rule:

```c#
protected const int MAX_PAGES = 10000;

[SetUp]
protected void Init()
{
  this.mBook = new Book("CIL Programming: Under the Hood of .NET",
    "Jason Bock", 360);
}

[Test]
public void CheckPageCount()
{
  Assertion.Assert(this.mBook.Pages > 0);
  Assertion.Assert(this.mBook.Pages < MAX_PAGES);
}
```

The test will succeed in this case, but if I create a `Book` in `Init()` with a page count of -40, `CheckPageCount()` will fail.

The issue is if I have an invariant condition set up on any `Book` object that specifies the valid range of `mPages`, why write up a test for it? If I'm guaranteed that I can never create a Book that will have an invalid page value, then why write the `CheckPageCount()` test?

Again, I'm not knocking unit testing. But if you have DBC, you explicitly know what an object can and cannot accept. This assurance would seem to make some unit tests unnecessary.

> Published: 11.20.2002 08:30:00 AM CST