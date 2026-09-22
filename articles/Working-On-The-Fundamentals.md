---
title: Working on the Fundamentals
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Working on the Fundamentals

My desire to work as a software developer wasn't immediate. I had a fairly strong interest in working with computers and programming, but I never spent a large amount of my free time writing code. Coming out of college, I would have defined myself as a "hacker" in that my code was smashed and taped together into something that worked, but it was by no means elegant.

Two specific cases come to mind. When I entered graduate school, I wasn't sure what I really want to focus on. I was in the electrical engineering program, but I was a little disoriented going into my first semester. I didn't know out of the gate where I wanted to focus my studies. I finally figured out what I wanted to do with [my master's thesis](https://epublications.marquette.edu/theses/3962/), which was to create a program that could calculate pi to 5,000 digits using something called a [number theoretic transform](https://en.wikipedia.org/wiki/Discrete_Fourier_transform_over_a_ring#Number-theoretic_transform). I got it to work, but looking back on that code, it was a large amount of C that violates a lot of programming principles that I didn't know about back then, such as:

* Duplicated code: Functionality was copied and pasted all over the place, rather than shared in common methods. 
* Undocumented values: Magical constant values appeared out of nowhere without any context.
* Odd naming conventions: Variable names were cryptic. (I used a fair amount of comments, though, so now that I look at it, it's not completely unreadable!)

Not having those principles in place, I come to my second case, which was my first job out of college. I spent a year or so work on an application that would allow people to apply for jobs when they visited one of my employer's locations. Not only was I still repeating my same mistakes from before, but I was also performing these bad practices:

* Large implementations: I remember my functions contained numerous responsibilities, which made it difficult to understand what was going on when features didn't work as expected. 
* Bad performance: I really didn't know how to approach coding with speed in mind. Profiling and diagnosing applications for hot spots was completely off my radar. 

Basically, for my first year as a professional developer, I struggled. I got things done, but I would get frustrated with my efforts. I was embarrassed when users would wait seconds, if not minutes, for the application to finish an activity. It was getting harder and harder to add features and fix bugs. I eventually came to the point where I had to figure out what makes code "good".

Fortunately, around that time I discovered ["Code Complete"](https://www.microsoftpressstore.com/store/code-complete-9780735619678) by Steve McConnell. He emphasized consistency with naming conventions. He emphasized using small functions, breaking up longer one to make it easier to follow the flow. I had a hard time putting that book down – it changed my mindset about code. Slowly, over time, I started to apply these principles to my code. It didn't make things immediately better; in fact, arguably I'm still learning how to write code better. But I noticed that I could navigate my code easier. It was simpler to debug. It was (and still is) an evolutionary process, but I was seeing positive results.

If there's one lesson that I learned as a junior developer that I would share with others, it's this: learn the fundamentals. That sounds like such a basic piece of advice, but working with people with little to no experience has shown me that these fundamentals are, more often than not, non-existent. Even in 2016, where computers are much faster and more reliable, the developer tools are easier to work with, it's still critical to write code where you've put forth the effort to make it easy to read and understand. The structure is consistent with other pieces of code in the application, there's a substantial amount of testing in place (I'll revisit that point in my next article), and it doesn't take a lot of effort to alter the code when new features come into play. Neil Peart, in his book, "Far and Near: On Days Like These" describes the underlying quality to his music and writing with the phrase, "care has been taken":

> "That could be a decent metaphor for life - investing your time with care, selectively, to the work, play, and people sharing your life. Also investing that time with care emotionally - doing those things, living those things, as well as you can, for yourself or for others."

As a software developer, it's a timeless, fundamental truism that well-structured code is easier to work with over time. That doesn't come for free, though – we need to have that emotional investment in our craft. If we dismiss our efforts with, "no one will really use this part of the application" or "we're not striving for perfection, we just need to get something out the door" we'll end up with a pile of broken, unmaintainable code that no one wants to get near. Getting a solid core in place and having good habits in place will reap rewards in the long run. Take the time to read programming books like the ones listed at the end of this article. In the end, you'll able to see that "care has been taken" and you'll reap the benefits of a clean, manageable code base. Until next time ... happy coding!

## Recommended Reading:

* "Code Complete" by Steve McConnell
* "Refactoring" by Martin Fowler
* "The Pragmatic Programmer: From Journeyman to Master" by Andrew Hunt and David Thomas

> Published: 04.18.2016 08:22:10 AM CST