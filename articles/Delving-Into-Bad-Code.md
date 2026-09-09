---
title: Delving Into Bad Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Delving Into Bad Code

The madness must stop! Over the years, I've noticed the following general rule of thumb take form over and over on the projects I would work on: while the demand for developers has gone up, the quality of code has gone down. I've been on two VB-based projects that were already in production, yet they were extremely unstable and prone to run-time errors. The fact that both didn't have any error handling didn't help, and the underlying code bases were terrible as well. In one app, goto statements were peppered throughout the one main method that did all of the processing. In another, the developer decided to use data binding along with ADO Connection objects, making database access a real mess. In both cases, I had to spend a lot of time refactoring the code base to get it into a stable state while the users were demanding new functionality. This isn't what I consider fun. 

Now, that doesn't mean that I think every developer should be fired. I've met and worked with developers who I have a great deal of respect for. They spend time perfecting their skills. They learn new things. They're open to new ideas, and they're willing to share their ideas with others. Nor do I think I'm a coding virtuoso. It's easy for me to think of projects in the past where I know I'd design them differently now. I also know that I've written code in the past that would make me cringe if I had to support it. Such is the educational experience called a "career." 

Unfortunately, though, I have seen a lot of people who just don't seem to mind that their code is unmanageable. They could care less if their classes have 120 methods (yes, I've seen this). They never (or very rarely) test their code, and when it breaks you'll hear the rally cry of "Hey, it compiled!", or, "It works on my machine!", or, "they shouldn't be passing in that value for the third integer argument anyway, the moron!" In short, if it's working, that's good enough, and hopefully someone else will maintain it. 

Granted, working conditions usually aren't the best. Cubicle hell is all too common for the developer, and this isn't right, period. Developers need their own space to shut a door and think about the problem at hand. They shouldn't have to be in an environment where they constantly hear background noise or hear the person in the next cube gabbing away on the phone. But sometimes, even in the best of conditions, bad code rears its ugly, stinking head. 

## So What Is Bad Code?

To me, bad code is kind of like pornography: You can't define it but you know it when you see it. This isn't a good enough answer, though, and that's why I've decided to start adding articles on angryCoder that demonstrate what I think bad code looks like. I want to be able to figure out just what makes bad code ... bad code. Each article will start with the question, "What's Wrong With This Code?" followed by the code snippet. Take some time to read the code and see if you come to the same conclusion that I did. Sometimes, though, it may be hard to do so, as I need to add more contextual information to the problem. Once I've explained why I think this is a good example of bad code, I'll give an alternative solution. 

Note that the code examples are in different languages, ranging from Java to C++ to SQL. Hopefully, even if you don't know the language at hand, you'll be able to see where the problem lies. Bad code is not language-specific – it's developer-induced. 

## Disclosing Code

As I work on many different projects, I inevitably see bad code. However, as the code I see is usually written for a company, I have to be careful not to disclose any "trade secrets" in the code I show here (like creating an ADO recordset - damn, that's hard!). I have done the following things with all of the code examples: 

* I've trimmed the code down to its essentials.
* I've changed variable names from their original names if these names contained company-related indicators.
* Any other indicators (such as comments with programmer's names, etc.) have been stripped to protect the guilty.

Rest assured, the code I've posted won't be used to create artificial intelligence. But I respect the agreements I'm under, so I've made all attempts to hide what the code does and where it came from. Remember, though, this is bad code. Would you want to claim that this code is yours? 

## Preventing Bad Code

Martin Fowler's book, "Refactoring," and Steve McConnell's book, "Code Complete" are excellent examples of how to create good code and improve on it. I wish more people would read these books. Bertran Meyer's "Object Oriented Software Construction" is also very good reference, especially the sections on Design By Contract (you'll never want to code without preconditions and invariants!). 

## A Final Thought

I have to admit, I'm a bit concerned about writing a series of articles on bad code. It may appear to come across as negative. At the same time, that's one of the aspects of angryCoder – speak your mind! This is one aspect that I know frustrates many developers; Dealing with code that makes you wonder, "How did this ever get into production?" However, I'm hoping that I can shed a positive light on the bad code I show. It's easy to show how annoying something can be, but if you don't propose solutions, you're just wasting time. I'm hoping that I can balance the bad with the good. I'm trying my best to learn from the mistakes I make as well as from the mistakes others have made that I have to fix. I'm hoping I can communicate that wisdom to you effectively. 

> Published: 04.05.2001