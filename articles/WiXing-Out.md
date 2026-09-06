---
title: WiXing Out
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WiXing Out

Yesterday I decided to get the build script done. One of the requirements is that the deployment part of the build creates a MSI via WiX. I've never used WiX before, so I tried to go down the route of creating a MSI via a VS .NET Setup project, decompiling that via dark to get the .wxs, and then use candle and light to get the MSI. That didn't work :(. After some Googling, I found out that taking the decompiling dark route is not the way to go. So I ended up creating my own streamlined .wxs file, and then everything worked as expected. WiX seems like a cool set of tools to use during the Nant build process, and I hope I can invest more time into its' intricacies.

> Published: 07.23.2004 08:18:20 AM CST