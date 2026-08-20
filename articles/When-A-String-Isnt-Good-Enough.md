---
title: When A String Isn't Good Enough
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When A String Isn't Good Enough

Today I found this code in our code base:

```c#
string value = ConfigurationManager.AppSettings["AConfigurationValue"].ToString();
```

I guess getting a `string` back (which is what the indexer returns) isn't good enough - we need to call `ToString()` on a `string` to get the right `string`.

Sigh ...

I wonder if there's an FxCop rule to warn you if you're calling `ToString()` on a string ...

> Published: 08.22.2007 09:19:02 AM CST