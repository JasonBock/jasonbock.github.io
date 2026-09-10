---
title: How Requirements Help Make Vacations Successful
layout: default
---
| [Home](https://jasonbock.NET/index.html) | [Biography](https://jasonbock.NET/Biography.html) | [Speaking](https://jasonbock.NET/Speaking.html) | [Articles](https://jasonbock.NET/Articles.html) | [Books](https://jasonbock.NET/Books.html) | [Music](https://jasonbock.NET/Music.html) |

# How Requirements Help Make Vacations Successful

Back in February of 1997, I changed jobs and started working with a consulting company. My first client was an industrial manufacturing company where thousands of technicians would work in the field installing and servicing their equipment. To enter their time sheets, the techs would use an Excel spreadsheet to fill out their information for the week and submit it to the accounting department. The people in accounting would re-enter the technicians' information into the financial application.

Needless to say, this was inefficient for many reasons, and they had decided to create their own internal Web application to allow technicians direct access to the back-end systems.

Remember, this is 1997, when Netscape was the browser of choice and using Perl CGI was an accepted Web development approach.

I was responsible for creating a VB application that allowed accounting employees to enter time for technicians who couldn't access the Web application, along with creating payroll and labor files every week for other various back-end systems. The other developer on the project handled the website (and yes, he used Perl).

Needless to say, the payroll file that was created on Monday was pretty important to get right and done on time. Unfortunately, we were working in an environment with a SME who, while he was a good person at his core, was hard to work with at times and often spoke before he thought through the ramifications of his statements. Requirements were virtually non-existent, and testing ... well, if we thought it was correct, we shipped a new version. You can imagine that this didn't work very well, and in September things came to a screeching halt.

## Mistakes Were Made

I was married near the end of that month. The Thursday before I went on vacation, the team was going to ship a ton of new features in the application. We were concerned that there would be issues, and while the Web developer handling the Perl side of things could work his way around in VB, he wasn't the one handling that area on a day-to-day basis.

We tried to do more testing before I left, and that gave us more confidence that the system would still work as expected. However, as you can imagine, there were problems, the biggest one being that creating the payroll file didn't work.

Again, I'll stress that this happened in 1997, a time when I didn't have a cell phone or a laptop with me such that someone could easily get a hold of me and yell at me to fix this ASAP. (Quite frankly, I would've ignored that anyway -- I was on vacation, and you're supposed to disconnect on a vacation!) Thankfully, the other developer on the project was able to figure out what the problem was and push a new version, but the fact remained: We still failed.

When I got back, I found out that people were not happy. I felt I was going to be fired. My stress and anxiety levels went from 1 to a 9 on a scale of 1 to 10 quite rapidly. But, out of this chaos, our project manager stayed calm, and used this experience to expose the real culprit: requirements.

We needed to slow down and spend the time to define all of the rules in detail. We had to be clear on any alternative paths that could be done and how they worked. We also had to create a thorough test plan that had to be accepted by various members of the team before we agreed to push new code to production.

## Requirements Help!

It took about two to three months to put this formalization in place, but once we did, our next release was stable. So was the next one, and the next one, and soon the business gained confidence that we could deliver new features without breaking the system. Things ended up in such a good state I was asked to work on other applications at that client, either to help them out and get them back on track or start them from scratch, applying our approach right from the start.

It's common these days for people to talk about the "minimum viable product." There's a mentality that getting to market faster than your competitors is essential, even if you know there are bugs in the product or you had to cut corners to make everything work.

To an extent, I understand this view and there's validity to it. But if you can't even define what the application is supposed to do at a fundamental level, you're screwed. At some point, all it will take is just one line of code to do a lot of damage (that's literally all it took to crash that payroll file processing code).

Having clear, well-defined requirements is a basic that every application must have. This is an evolutionary, iterative process, but it's one that must be in place for a software project to be successful. Until next time, happy coding!

> Published: 06.01.2016