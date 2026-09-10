---
title: Shrug Emojis, Console Applications, Fonts and Unicode
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Shrug Emojis, Console Applications, Fonts and Unicode

Synopsis: Sometimes, a fun challenge can lead one down paths that can frustrate and enlighten the traveler. Here is a story of what I thought would be an easy thing to do in code, but ended up being more of a trial than I expected.

## Having Fun with Exception Messages

Recently I was writing code for another blog article, and I got to a point where I wanted to write an `if-else` statement. If the `if` condition was `false`, the `else` was considered handling an error condition, but I really didn't know what the right approach was. Do I return an error object of some kind, or should I throw an exception, or do nothing? I ended up thinking that I should throw an exception, but which one? Since I was going to support a specific operation, I reasoned that `NotSupportedException` was the right one.

Then my mind started to wander. `NotSupportedException` or `NotImplementedException` feels like you're basically giving up on the caller. "Too bad, so sad" is the message you're conveying. Like you're shrugging your shoulders and broadcasting a big "meh".

That made me think ... you know, maybe the right message to use with `NotSupportedException` is the shrug emoji! As a joke, I wrote this:

```c#
throw new NotSupportedException(@"¯\_(ツ)_/¯");
```

I ran my console application, and the result was less than promising when I raised that exception:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-1.png "Shrug Emojis")

Oh boy. That does not look right! I had the feeling I was going to head down a Unicode path of despair and sorrow.

## Console Output Woes

To be honest, this wasn't a pressing issue. I created that message for the exception as a joke. But, I was also curious to see how I could get the right text to show up. I mean, in Visual Studio, I was seeing the shrug just fine:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-2.png "Shrug Emojis")

In fact, if I ran my code under the debugger, I'd see the shrug like this in the *Exception Unhandled* window:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-3.png "Shrug Emojis")

Furthermore, in the *Locals* window, it looked like this:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-4.png "Shrug Emojis")

So why wasn't it looking right in the console window? As I said before, my guess was that it had something to do with some of the characters within the shrug text being Unicode characters. However, I have little experience with the ins and outs of Unicode, so I decided to throw out a challenge to other Magenic developers by posting this to our .NET channel on Teams:

> Get this exception with its message to show up correctly in a console window.

I think developers need to do this far more than they do. Developers have this inner desire to solve every issue and problem they run into. That's a great quality to have, but, in a weird way to show just how smart they are to everyone else, sometimes they try to figure it out without any help. The older I get, the more I realize just how little I know. My fear of looking "stupid" by asking a question has lessened dramatically.

Also, while this may seem like a trivial thing, I still thought it had relevance. I've always used console applications as a simple way to try different approaches in code. However, I've never been a user of the command line either, primarily because I've been a .NET developer for a long time. Frankly, that world hasn't been very command-line driven. But it seems like that's changing, so know how things work from the console window can't be a bad thing. And who knows – maybe the knowledge I gain by figuring out this goofy problem can be beneficial to myself and/or others at some point in the future.

As it turned out, I got a couple of suggestions that didn't solve the problem, but started getting me down a decent path.

## Unicode Values

Before we continue, let's look at each character in the message and see what their values are:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-5.png "Shrug Emojis")

It turns out that there are two characters that are problematic. The first one is obvious: it's the "face" character in the middle. It's called Katakana Letter Tu. The other one is the "hands" that start and end the string. It's called Macron. Both characters are outside of the ASCII values that range from 0 to 127. However, .NET strings can easily represent these characters because .NET strings are Unicode strings. Given this, that pretty much ruled out the underlying data type as being the issue as simply creating a string based on these characters. The first suggestion was to change the encoding of the output stream to this:

```c#
Console.OutputEncoding = Encoding.UTF8;
```

When I did this, things got better, but it wasn't right:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-6.png "Shrug Emojis")

The "hands" look right, but the "face" is now a blank rectangle (at least it's no longer a question mark). I tried other encoders, like `Encoding.UTF7`, but in each case, the output wouldn't be perfect. Some didn't work at all, like `Encoding.UTF32`. That would cause an `IOException` with the message, "The parameter is incorrect." At this point, I thought, changing the encoding was part of the answer, but not all of it.

## What's a Code Page?

Another suggestion was to change the code page of the console window. A code page is ... well, it's kind of complicated. You can read more about them [here](https://learn.microsoft.com/en-us/windows/win32/intl/code-pages) and [here](https://en.wikipedia.org/wiki/Windows_code_page). Essentially, here's how a code page is defined (to quote from the second link):

> Windows code pages are sets of characters or code pages (known as character encodings in other operating systems) used in Microsoft Windows from the 1980s and 1990s. Windows code pages were gradually superseded when Unicode was implemented in Windows, although they are still supported both within Windows and other platforms.

After reading that, I'm not quite sure why it would help to change the code page, but I was already kind of stumbling around in the dark a bit, so why not try it? To do this required a p-invoke call:

```c#
[DllImport("kernel32.dll")]
private static extern bool SetConsoleOutputCP(int codePage);
```

Someone suggested using the Japanese code page value, 932, given the "smile" character in the middle. As before, this did something, but I still had issues:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-7.png "Shrug Emojis")

Now I get the Yen character on one side of the "face" character.

I tried a couple of other things with code pages, but it quickly felt like this was the wrong approach. Even worse, I seemed to get cmd.exe stuck in the 932 code page value, no matter how many times I called `SetConsoleOutputCP()` with 437. It took a regedit search for "cmd.exe" to find an entry that had a `CodePage` key, which was set to 932. Setting that to 437 got things back in order.

Therefore, I quickly moved away from code pages being part of the solution. I could be wrong on trusting my intuition here, but it seemed like it wasn't going to solve my problem and was actually causing other annoyances. There has to be a better way, right?

## Fonts to the Rescue!

As it turns out, there's an easy way to solve the issue. I just had to broaden my query a bit more by heading to the Internet! After a couple of searches on StackOverflow, I tried doing this: change the console's font to something like NSimSun:

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-8.png "Shrug Emojis")

When I did this (in concert with changing the encoding value), viola! It worked!

![Shrug Emojis](https://jasonbock.net/images/Shrug-Emojis-9.png "Shrug Emojis")

## Addressing Requirements

At this point, I would argue that I met the challenge. I have to change the font in my console window to something that looks decent, but I like Consolas. I'd much rather use my font of choice. I thought of looking into changing the font when the application starts and then reset it when the application finishes, but developers are very picky about their setup; doing something like that feels rather evil.

There have been times in my career where I read requirements (yes, those actually do exist), and there is a degree of ambiguity within them. I'll try to get these questions answered before I get too far, but even then, I may come to the end of the task and think that I've finished it, only to find out the user wasn't happy with the end result. Specifically, in this case, I could say, "yep, it works!" but the user could say, "hey, I don't want to use NSimSun! And I can see that text just fine in Visual Studio and that font is Consolas! Fix it!!" I wish I had a solution that used Consolas in a console window and had the shrug text showing up just fine. But I don't. If someone has a solution that doesn't feel insanely hacky I'm all ears!

## Conclusion

We came from a simple, goofy "what-if?" problem to a solution that doesn't quite seem to satisfy. I've heard the phrase, "perfect is the enemy of the good", and I have a hard time with that statement. I feel like it's used as an excuse to get low-quality products out the door in the name of the "good" being equated to "releasing". That said, in this case, I was able to come up with a solution that work, though it's not quite what I want. I can say that I learned a bit more about Unicode and fonts and console apps, though, so, it's not all bad, right? As a wise person once said, ¯\_(ツ)_/¯.

> Published: 09.18.2017