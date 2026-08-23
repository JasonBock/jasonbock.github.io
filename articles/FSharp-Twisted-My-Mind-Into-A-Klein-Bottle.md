---
title: F# Twisted My Mind Into a Klein Bottle
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# F# Twisted My Mind Into a Klein Bottle

I was reading the contents of chapter 5 (get 2-7 here), and I started reading the "Generic Algorithms through Explicit Arguments" section.

It took a **long** time before I understood what I read, and by that time I think my brains fell out of my ears. But this blurb really make it clear:

> One of the remarkable features of functional programming with F# is that it makes generic programming practical even when the types involved are not explicitly related. For example, the generic algorithm above shows how reusable code can be written without resorting to relating the types involved through subtyping: the generic algorithm works over any type, provided an appropriate set of operations is provided to manipulate values of that type. This is a form of **factoring by functions**. Object-oriented inheritance hierarchies are a form of **factoring by library design**, since the library designer decides the relationships that must hold between various types ... F# makes factoring by functions extremely convenient, and given the simplicity and flexibility of this approach it is recommended as the approach to use when writing applications and proto-typing frameworks.

Basically he's describing this code:

```f#
let hcfGeneric (zero,(-),(<)) =
  let rec hcf a b =
    if   a=zero then b
    elif a<b then hcf a (b-a)
    else hcf (a-b) b 
  hcf

let hcfInt = hcfGeneric (0, (-),(<))
let hcfInt64 = hcfGeneric (0L,(-),(<))
let hcfBigInt = hcfGeneric (0I,(-),(<))
```

F#. I really gotta wonder if it's **the** .NET programming language these days...

> Published: 01.25.2007 09:23:57 PM CST