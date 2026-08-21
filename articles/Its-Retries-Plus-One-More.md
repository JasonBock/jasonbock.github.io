---
title: It's Retries ... Plus One More
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# It's Retries ... Plus One More

I've been tasked with seeing how our WCF services will work with MSMQ and failures. Basically we want to make sure that messages are preserved even if the database fails. I created a simple test scenario and I was able to figure out what happens (we'll basically have to restart the host when it faults but the messages stay in the queue and will be retried when the host starts up again and that's OK). The documentation was a little ... odd. Check out what it says for `ReceiveRetryCount` (note the bold text):

> ReceiveRetryCount: An integer value that indicates the maximum number of times to retry delivery of a message from the application queue to the application. **The default value is 5**. This is sufficient in cases where an immediate retry fixes the problem, such as with a temporary deadlock on a database.

Now check out what the formula says for "the maximum number of delivery attempts":

> **(ReceiveRetryCount + 1)** on Windows Server 2003 and Windows XP.

So that's why I saw six failures and not five - got it!

Sorry, but that documentation is confusing as hell. If the default is five, then why is there a "++" feature? This seems really odd to me.

> Published: 05.29.2007 09:57:51 AM CST