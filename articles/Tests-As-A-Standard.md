---
title: Tests As a Standard
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Tests As a Standard

Back in 2003, I was on a project where we did everything according to Extreme Programming (XP) principles. We reviewed user stories, we did short iterations, and we did pair programming (I think this was the only project I've done where we followed this discipline). In short, it was a vastly different experience than what I was used to.

I was willing to try new approaches -- in fact, it was good to be on a project that was committed to actually have some kind of management and guidance in place! However, I was not on board with unit testing. To me, it was a waste of time. Why write tests if you ensure that your code is good through continuous code reviews? I figured that we're working with someone else who is looking at your code constantly, and we're always testing the code by running the application, so why do extra work? But the team wanted to do unit testing, so I (grudgingly) wrote them as well.

For the first couple of months, unit testing didn't resonate. I was doing them as a rote exercise, but I didn't understand the benefit. Then, one day ...

## Seeing the Benefits

I came into work and started working on a new feature. At some point, when I ran all the tests, I saw that a couple tests failed. This was completely unexpected. Why would these tests fail? They have nothing to do with the change I was working on! But after further investigation, I realized that my new code broke accepted behavior.

Now, to be fair, the tests we were writing were arguably not "unit" tests. They probably had more in common with integration tests as we didn't isolate as well as we should. But this experience of failure caused a rapid change in my view of automated testing. I realized it was easy to run a suite of tests without incurring any cost in time compared to manually running test cases. Furthermore, by writing and running these tests, I had a better chance of uncovering issues as I modified the code base while my changes were still on my machine.

This enabled me to "fail fast," and knowing that something was broken on the machine I was developing on meant I could fix the issue quickly, rather than waiting for QA or, even worse, users to notice the bug.

## Testing Isn't Optional

Since that inflection point in my thinking around unit tests, I've always done unit testing on projects that I've been on. I've grown in terms of understanding the different kinds of tests that can be run (i.e. unit, integration, end-to-end, etc.) and the tools and frameworks that can be used in tests.

In general, I've observed that projects with unit testing in place drive developers towards success and stability. I've also experienced projects where a code base has been around for a long time and there are no unit tests to be found. Writing tests in those cases is a difficult endeavor. Furthermore, those code bases generally are difficult to navigate and hard to add new features. They're tightly-coupled, the amount of code within a class or methods can be quite long -- in short, they're usually a mess.

In 2012, I was on a project for a client in the financial space, writing an application for loan processing. This application had numerous business rules and validation requirements, along with multiple paths with the loan flow depending on different choices made in creating the loan. It was critical that we got all of these cases right.

Once the project was released, we had more than 6,000 unit tests. The end result was an application that was received positively by the users, and the client also gave an excellence award to the business group responsible for managing this new application, something that they told me they had never received with any products they had created before. Given the size of this client, that said a lot about the stability and reliability of the software. This, in part, was driven from developers committed to verifying that their code worked as expected.

I'm convinced that it should be a requirement that developers write tests. They provide a higher level of confidence for developers that their code actually works the way they think it should. They can help uncover issues with new features that are added over time.

It's not a panacea: It takes time to write and maintain those tests. But any successful software project requires comprehensive testing, and that time investment must always be made. It's cost-effective to write those these as they will reduce the amount of bugs and issues over the lifetime of an application.

Until next time, happy coding!

> Published: 07.21.2016