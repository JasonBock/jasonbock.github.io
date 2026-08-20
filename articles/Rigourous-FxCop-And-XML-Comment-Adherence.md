---
title: Rigourous FxCop and XML Comment Adherence
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Rigourous FxCop and XML Comment Adherence

This morning I spent some time looking at the code base, specifically at our XML comments (or lack thereof) and our FxCop violations.

Ugh.

And I mean, **UGH**.

I'm convinced that if you really want to have XML comments and integrate Code Analysis in your project, you have to address the issues at the level of compilation error if you want to take it seriously. If you want to ignore the stuff, fine, but trying to go back and refactoring the code after it's been percolating for months is equivalent to trying to fix code that has lots of compilation errors. It's an exercise in futility. It's extremely difficult to understand the intent of what the developer(s) was/were trying to do before you got there.

I'm an advocate of XML commentary and Code Analysis. But it has to be treated as seriously as compilation errors if you want to work effectively. Otherwise, just turn them off. But I wouldn't recommend that, especially for Code Analysis. I always find lots of interesting design issues with it on. Even if I don't agree with it, it still makes me think about why the code was written the way it was.

> Published: 07.26.2007 08:43:03 AM CST