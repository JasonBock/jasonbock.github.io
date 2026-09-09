---
title: A Critical Decision
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Critical Decision

## What is Wrong With This Code?

Language/Tool: Visual C++ 

```c++
unsigned char* ConvertData(unsigned char* pbyTo, 
  unsigned char* pbyFrom, long lLen)
{
  CRITICAL_SECTION critSec;
  InitializeCriticalSection(&critSec);
  EnterCriticalSection(&critSec);
    
  unsigned char* pbyRet = pbyTo;
  unsigned char convertTable[256] = {0x00, 0x01, ...};
  
  while(lLen-- > 0) 
    *pbyTo++ = convertTable[*pbyFrom++];
    
  LeaveCriticalSection(&critSec);
  DeleteCriticalSection(&critSec);
    
  return(pbyRet);
}
```

> Note: I've taken out the `convertTable` initialization for space considerations.  

The `ConvertData()` method is meant to take a bunch of ASCII data and convert it into EBCDIC format. If you're not familiar with this format, don't worry - it's not necessary to know about it in this situation. 

One of the odd things that I thought about when I saw this method was that it was the third time that I found an ASCII-to-EBCDIC conversion method in the system (so much for code reuse). Why this was done is anyone's guess. Maybe three different people didn't like what the other two did and added the conversion method where they wanted it. Maybe nobody knew that the other methods existed. Whatever the reason, it's a sure candidate to get refactored if time allows for it in the future (i.e. create a class that handles these conversions and use it everywhere within the system). There is simply no need for three conversion functions when they are all doing the same thing. But there is a subtle time-waster in this method that needs to be addressed as well. 

It may be a bit tricky to spot, especially if you've never done multithreaded applications before. Let's back up for a second and talk about what a critical section (CS) is. A CS is used in a Windows application as a locking mechanism between threads. One thread creates the CS (`InitializeCriticalSection()`) and/or acquires the CS (`EnterCriticalSection()`), uses the data that is protected by the CS, and then releases it (`LeaveCriticalSection()` - it may also delete it via `DeleteCriticalSection()`). That way, if another thread comes in to use the protected data, it'll have to wait until the first thread is done. There are other locking mechanisms available in Windows, such as mutexes and semaphores. CSs are the cheapest of them all, but they only work within a process - that is, you can't create a CS in process A and then use it in process B (note that you can do this with a mutex). However, in this case, that's not the problem. 

> Note: Multithreaded development can be quite difficult if you're not careful. For a great resource, I'd check out "Win32 Multithreaded Programming", written by Aaron Cohen & Mike Woodring. You'll find everything you need to know to create safe, efficient Win32 multithreaded applications (I hope they update this for .NET in the near future).  

It appears that the code is trying to make sure that if the client calls `ConvertData()` more than once with the same `pbyTo` buffer, the data won't be clobbered with multiple writes. However, if this is the case, the declaration of `critSec` is in the wrong place. Let's say I called `ConvertData()` with `bufferOne`. Now I pass `bufferOne` into a separate thread, and call `ConvertData()` with that same buffer. The first time `ConvertData()` is called, I enter a CS - let's call it CS1. Then, in the second thread, I create another CS, and enter that one - call it CS2. Once that second thread acquires CS2 and starts to convert data, my destination buffer becomes mangled.

Why? I created two different CSs - neither one prevents the multiple-write issue I talked about above. To me, this code demonstrates that the implementor was trying to prevent a possible multiple-write scenario against the same buffer. So s/he looked up some information on locks in a Windows multithreaded application, found out that CSs are cheap, and decided to throw them in, thereby completely ensuring the safety of the incoming data for all time. In other words, the implementor didn't know what s/he was doing, nor did s/he determine if this method really needed to be thread-safe. 

## Possible Solution

The simple solution is to move `critSec` outside of `ConvertData()` and make it a class member variable: 

```c++
CRITICAL_SECTION m_critSec;

unsigned char* ConvertData(unsigned char* pbyTo, 
  unsigned char* pbyFrom, long lLen)
{
  InitializeCriticalSection(&m_critSec);
  //  And so on...
}
```

In fact, it makes more sense to move the `InitializeCriticalSection()` call to the class constructor and the `DeleteCriticalSection()` call to the class destructor, but I'll defer that discussion, as I think the CS should be removed altogether. The system code never calls `ConvertData()` from two different threads with the same destination buffer, so a CS is not needed. Even if the same destination buffer is being used in this fashion, I'd have to wonder why someone would do that in the first place. If you have two different source buffers, why use the same destination buffer in two different threads? If the goal in this case is to save on memory, then you'd need to synchronize access anyway, and even if you did it right (i.e. put the CS outside of the method as a class member variable), you might as well process the two source buffers sequentially in one method as you'll avoid thread context switching overhead.

## Should It Be Changed?

In all of the articles that I've demonstrated so far, the code needed to change, even if it worked a given percentage of the time (at least they all "compiled"). In this case, however, I wondered if it should be changed at all. Why? There's a couple of reasons. One, there's time and money involved whenever you make a change. As much as I think removing the CS does not affect the behavior of the system, I would need to retest the system after I made the change. It's a simple change, and I know that nothing would be affected, but I also believe that whenever you make a change, you run a regression test. The other reason is that the system is not being "harmed" by creating, using, and destroying the CS. Granted, you're wasting CPU cycles by doing this, but if there are other pressing issues I'd leave this be for now (and in this system, there were more important problems to solve, trust me!). 

What I'm trying to say is this: Don't just cut code that you think is unnecessary. No matter how trivial you think the change is, it may be costly. Money is involved in the time it takes for you to look at the code, determine that it can be removed, removing it, recompiling, and retesting. The resultant cost will vary depending on the situation, but it's never free. 

I'm not arguing that you should leave bugs in the system. Nor am I advocating bad code; far from it. But far too often I've heard the battle cry of, "that's a simple change," only to see some poor developer sweating profusely to figure out why the system doesn't work anymore after the deletion was made. Even if you know in your gut that a piece of code is completely worthless, don't remove it unless it's agreed that the task should be completed. A "five-minute" task can easily end up being a five-day marathon. It's worth reviewing the change with your peers - spending more time ensuring that the change is valid is indeed time well spent. 

Out of curiousity, I decided to determine just how much time this code was chewing up by creating unnecessary CSs. I wrote a small Windows test app that had two methods. One was called `MethodWithCS()`, and it would initalize a CS, acquire it, release it, and then delete it. The other method, `MethodNoCS()`, wouldn't do a thing - it contained no code. After calling each of the functions the same number of times, I found out that the non-CS function was just over 60 times faster than the CS function (if you try to recreate this test, note that your results may vary). So in this case, you do gain a lot by removing it. There's still one thing to keep in mind, though. If this function is only being called "sporadically" (i.e. once per month), then the time and money that it takes to remove it may actually be more expensive than it is to just leave it in. However, if the method is constantly being invoked, then refactoring it should be put high on the task list, as the users will gain from the reduction in computing time. 

## Should It Have Happened At All?

As a side note, another developer on the project sent this piece of code to a friend of his, just to confirm our suspicions that the CS was not needed. This was his reply: 

> For the good of society, please dispose of the person who wrote this code immediately.
>
> Sincerely,
>
>  SomeDeveloper (name hidden)
>
> P.S. A critical section on the stack would be considered worthless 

That sums it up nicely, doesn't it? None of this analysis would have been necessary if the original developer didn't add the CS in the first place. No matter what you're implementing, take the time to think about your implementation. Refactoring is helpful, but in my book, the best refactoring is when you don't need to do it because the original code base is solid. 

## Summary

1. Don't add functionality to your code base because you think it might help. Make sure that it's truly beneficial.
2. Don't remove functionality from the code base because you think it might help. Verify that it does.

> Published: 05.09.2001