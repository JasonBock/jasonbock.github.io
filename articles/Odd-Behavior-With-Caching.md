---
title: Odd Behavior With Caching
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Odd Behavior With Caching

Well, I thought my caching strategy was working, but it's not working on the production server. When I update the quotes.xml file on my local box, my local version of jasonbock.net clears out the cache and the file is reloaded again (which is what I expected). However, when I push the file to the web server, the cache is not refreshed. Now, I update my files via FTP - is there a potential issue that the `CacheDependency` object doesn't detect the changes because of the way I'm updating the file?

The really strange thing is that my blog history list is cached and refreshed when my rss.xml file is updated. This is working correctly. The only difference I can see so far is that I'm manually updating quotes.xml because it's not part of a blog update, whereas my blog is updated via the custom tool I wrote, which uses a freeware FTP component.

Any ideas?

> Published: 09.20.2004 08:18:33 PM CST