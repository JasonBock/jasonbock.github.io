---
title: Compilation Results From the CodeDom
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Compilation Results From the CodeDom

I'm playing around with the `CodeDom` classes, and I noticed that when a compilation occurs (e.g. by invoking `CompileAssemblyFromFileBatch()`) the resulting assembly is locked. That is, I can't delete it after compilation is done. Is there a way to close that file such that I can delete it after the compilation step if I want to?

> Published: 03.16.2005 07:53:01 PM CST