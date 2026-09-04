---
title: I Miss Concurrent Checkouts
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Miss Concurrent Checkouts

For the last year and a half, I've been on projects where anyone could edit any code file at any time. Only when you checked the file in were you responsible for merging stuff correctly (which good merge tools will make this fairly painless or at least less painful). This was so much better than anything I was on before in my career as it was always exclusive access, but usually the code bases were isolated to a few developers so it wasn't much of an issue. Having the ability to change a file when needed is an approach that's not without its own warts, but it allows the developers to have more freedom in evolving the system [1]. Now I'm on a project where only one developer can check out the file and leave others hanging in the breeze. I made some changes to two assemblies this morning, and since other assemblies rely on these assemblies, I needed to update them as well. Problem is, there's one code file left that needs to be fixed, and I can't get a hold of the developer who owns the file (phone calls, e-mail, nothing works, and don't say "walk over to his cube" because he's physically thousands of miles away). If I check in my changes the build will break and that stinks.

If you can completely control you code base and all of the assemblies that your code base relies upon, then you can get away with exclusive access. As soon as that assembly uses other assemblies, that plan becomes too rigid. The other assemblies may change and they'll need to change the depending assembly's code, but no one can because the file's locked! Again, I'm not saying that concurrent checkouts is painfree, but I feel frozen right now. I can't finish what I need to do (and that's changing 4 lines of code so a call to a static method is done on a different class - that's it!). And now others want to change the files I have checked out, but I can't check them in because I'll break the build. Argh!

[1] Having a build process that ran tests and having developers run tests before checkins occurred minimized any potential damage with the source code tree as well, but that's another story.

> Published: 01.19.2005 03:02:51 PM CST