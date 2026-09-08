---
title: .NET FX MSI Bug
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# .NET FX MSI Bug

I was watching some of the NY ceremonies this morning - it's still hard to comprehend that the Twin Towers are gone.

I got bit by the "netfx.msi bug" when I installed SP2 for .NET. I fixed it this morning by downloading the .NET redistributable EXE, pulling out dotnetfx.exe from that file, pulling out netfx.msi from that file, and then letting the indestructable Installer screen know where that MSI file was. The funny thing was the time estimate the installer gave me when it could finally finish what it needed to do:

![.NET Installer](https://jasonbock.net/images/DotNET-Installer.png ".NET Installer")

'nuff said.

> Published: 09.11.2002 08:48:00 AM CST