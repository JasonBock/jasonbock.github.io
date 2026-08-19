---
title: C# 3.0 Impressions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# C# 3.0 Impressions

I've been using C# 3.0 for a while now, and here's what I think of some of the new features. Don't expect long, drawn-out explanations with code examples; this is basically a brain dump :).

* *Lambda Expressions*. I **love** the syntax - it's so much cleaner than what it is in 2.0.
* *Type Inference*. I haven't used it much, but that's because I keep forgetting it's there. I'm a fan of consistency, so I don't know if I want to mix using "var" with typed variables or just say go with "var" all the time. But I do like that we have the option of letting the compiler figure out the type for us. I realize this is primarily used with LINQ - I'll get to that acronym in a bit.
* *Extension Methods*. Love 'em. I've written so many "helper" classes in the past, and the extension method approach cleans up the syntax so much. BTW here's a great resource on writing good extension methods. I realize the article is really about LINQ (again, I'll get the that acronym very soon!), but there's some excellent commentary on extension methods.
* *Automatic Properties*. I use them, but I'm not entirely thrilled with them. Generally I only use them if I have mixed-accessibility of a property that doesn't need initialization during object construction. Plus, I don't like how it mixes code styling with "richer" properties. I definitely like them more than I don't, but there's some aspects with this feature that I'd like to see changed in the near future.
* *LINQ*. See? I told you I'd get to this eventually! I still haven't used it. I love the idea of LINQ, but frankly I haven't had a pressing need to use it. It's probably because I haven't really forced myself to change my methods that have lots of foreach's with "where"-like checks to LINQ statements.

There. Overall I'm really happy with C#'s improvements. I've heard about some ideas for 4.0, and there's some things I'd love to see, like covariance and contravariance and great immutability support. There's also some things I'd love to see at the CLR level, primarily with Cecil-like features in Reflection, managed profiling and the ability to inject/modify code using a managed language.

Oh, one more thing I forgot about. I want tuples. Have you seen how tuples in F# make the `TryXXX()` patten so beautiful (go here and search for `TryParse`)? Amazing! Here's an idea of how it might look in 4.0.

> Published: 03.16.2008 09:05:59 AM CST