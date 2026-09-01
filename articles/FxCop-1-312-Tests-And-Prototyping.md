---
title: FxCop 1.312, Tests, and Prototyping
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# FxCop 1.312, Tests, and Prototyping

> Note: This post may ramble. You have been warned ...

Over the past week or so I've been working with FxCop - specifically, the 1.312 version. I can't say why just yet, not because I'm under some super-duper NDA secret, but I haven't finished all the stuff I'm doing with it yet and I hate it when people hint at working on cool things and then never produce. I have no idea if what I'm doing is cool nor do I know when I'll ever put the stuff on my site (the baby is coming in 3 weeks or less and I suddenly became involved in a bunch of other side projects) but it's fun and that's a good thing in my book. Anyway, I like the improvements that were made in 1.312 - the `Instruction` class has cleaned up my world from what I've seen in previous incarnations of the FxCop SDK [1], but it took me a while to get used to how things currently are before I could write some quality tests.

Generally, you hear the phrase, "write your tests first." I agree with that, but like any rule it's made to be broken ;). What I mean is, I started writing my tests, but I quickly realized that because I had no clue how the classes in the SDK worked, I had to stop for a while and do some prototyping to get comfortable with writing rules, visiting members, getting opcode values, and so on. Trying to write tests became meaningless because, while I know what I want to write and the tests to cover the implementations, I didn't understand the libraries I'd be using to write my implementations! So, off I went into prototyping land, trying something here, backing out something there, until I finally felt good about my level of understanding with respect to the SDK. Now I'm back to finishing my tests and making sure my rules work the way I think they should.

My point is, yes, you need tests, and you should try to write the tests first so you know what you're trying to cover. But don't feel bad if you have to spend a lot of time writing throwaway code so you understand how you're going to solve the problem and satisfy the tests. In my case, trying to pass the tests would have been a fruitless and frustrating endeavour until I grasped enough of FxCop's SDK to start writing some meaningful code. Trying to gain that knowledge just to pass test cases just didn't feel right.

[1] To the FxCop team, **PLEASE** produce some quality documents on the SDK! Something, anything!! Thank you.

> Published: 03.02.2005 12:27:56 PM CST