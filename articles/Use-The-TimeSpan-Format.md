---
title: Use the `TimeSpan` Format
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Use the `TimeSpan` Format

I was looking through our code base, and I saw this (ugly) configuration setting:

```xml
<add key="AuthorizationTicketTimespan" value="0|120|0" />
```

In this case, the configured value is 2 hours (120 minutes). What's ugly about it is that the developer created their own format string to configure a `TimeSpan`. Dude, that's so not necessary! `TimeSpan` has a simple format that's already defined: `[-]d.hh:mm:ss.ff`. The aforementioned configuration value could be changed to this:

```xml
<add key="AuthorizationTicketTimespan" value="02:00:00" />
```

And then we could use `TryParse()` on `TimeSpan`, thereby eliminating "duplicate" code in our code base.

I'd also recommend using this format rather than using an integer value for millisecond values in a configuration value. Using this one format across the board is consistent and easy to use.

> Published: 07.20.2007 10:13:14 AM CST