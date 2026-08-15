---
title: Even Bad Code Examples Still Need to be Right
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Even Bad Code Examples Still Need to be Right

Lately I've been spending some time with Code Contracts, Pex, and CHESS. My overall impressions with these tools/frameworks is positive, especially with Pex - that is one amazing tool! I've been having some issues with CHESS in that it's not working completely "as advertised" on the site but it's still impressive and I think it'll get better over time.

However, I had one big issue with one of their code examples, the `BankAccount` one. Here's the code, taken verbatim from the sample:

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Bank
{
  public class Account
  {
    private int balance;

    public Account(int amount)
    {
      balance = amount;
    }

    public void Withdraw(int amount)
    {
      int temp = Read();
      // oops, temp could become stale if we are
      // preempted here
      lock (this)
      {
        balance = temp - amount;
      }
    }

    public int Read()
    {
      int temp;
      lock (this)
      {
        temp = balance;
      }
      return temp;
    }

    public void Deposit(int amount)
    {
      lock (this)
      {
        balance = balance + amount;
      }
    }
  }
}
```

Now, this is bad code on purpose, because they're trying to show you how CHESS will find the bug [1] that it's talking about in the comments in `Withdraw()`. OK, that's fine, I don't have a problem with that. What I have a big problem with is this:

```c#
lock(this)
```

That is a **huge** no-no in concurrent applications [2]. Moreover, the issue with the code is where the locking is occuring, so developers may not pay attention to what the code is locking on. And this is the problem with code examples: either the people writing them don't take them seriously enough and/or people reading them look at it and go, "oh, that's how this 'expert' did it so I'm going to do it here."

Code examples are still code. And they're probably going to be used by those who don't have a lot of experience in software development. If anything, code examples should be shining examples of how to write code well. Even when you're showing bad code, only make it as bad as necessary.

[1] Unfortunately, I could never get CHESS to find it on my machine. I have a forum question open on this, but no answer so far.

[2] Surprisingly, CodeAnalysis does not find this. I'm guessing that the rule to know that you're locking on the "this" reference is not as easy as it seems in code, as the developer may have referenced "this" with a local variable, so you'd have to track back to see what `Monitor.Enter()` is actually locking on. But I could be way off on this too.

> Published: 04.01.2009 10:22:59 AM CST