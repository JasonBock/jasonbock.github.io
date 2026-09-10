---
title: Embrace Your Keyboard
layout: default
---
| [Home](https://jasonbock.NET/index.html) | [Biography](https://jasonbock.NET/Biography.html) | [Speaking](https://jasonbock.NET/Speaking.html) | [Articles](https://jasonbock.NET/Articles.html) | [Books](https://jasonbock.NET/Books.html) | [Music](https://jasonbock.NET/Music.html) |

# Embrace Your Keyboard

Recently I was working with another developer to solve a coding issue. Once we came to a potential solution, he moved to his computer and started to write code in Visual Studio ... driving virtually the entire process (other than typing code) by using the mouse. Adding files to projects, starting builds – all of it was mouse-driven.

It felt like I was watching paint dry while having paper cuts being applied to my fingertips. It was boring and painful at the same time. The act of grabbing the mouse, moving it up to the menu, drilling into a couple of menu items, compared to doing something like "Ctrl+W, S" (to get the *Solution Explorer* visible) adds up.

I don't claim to be a keyboard guru. But over the last year or so, I've forced myself to get familiar with keyboard shortcuts to see if it would make me a faster developer. I think it's paid off to some degree, because I've had other developers watch me when I code and say, "wait, what did you just do ... how are you done with that already?" Let's go through some tips and tricks I've learned to make my development experience quicker.

## Stop Using the Mouse!

Let me be clear from the get-go: The mouse is not evil. There are times where using the mouse (or a trackpad) to move around in a program is the right thing to do. When I'm trying to manipulate images in a program like Paint.NET, I'm using the mouse a lot to select, copy, and move content (among other things). Sure, I'll use keyboard shortcuts to invoke some features, but the mouse is the right thing to use in that application. But when it comes to a development environment, you have to consciously stop using it. So, first thing: move the mouse away from its normal resting spot. Consider disabling your trackpad. Do what you can to break that muscle memory and force yourself to figure out how to use your tool of choice purely from your fingertips.

Now that the mouse is lonely ... how you do your job without it? My development world is primarily in Visual Studio, so here's a couple of tips to use to drive that environment without pointing and clicking.

## Learn "Ctrl + Q"

I don't remember the exact version of Visual Studio when this feature was added (I think it was 2013), but it's one of the best features Microsoft ever added. It's such a simple thing, but it helps tremendously in discovering what features are available in VS and their associated keyboard mappings if they exist. It's called Quick Launch, and you invoke it from the keyboard with "Ctrl + Q". With just this one shortcut, you can discover a lot in a short amount of time.

I mentioned before that getting the *Solution Explorer* window open can be done with "Ctrl + W, S". (Well, at least that's the keyboard binding for me. Note that depending on your VS setup some of the key bindings I talk about in this article may vary from what you have.) Finding that out is as simple as typing "Solution Ex" in the Quick Launch window (which usually ends up in the upper-right hand area of Visual Studio):

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-1.png "Embrace Your Keyboard")

OK, that's 11 characters, but now that we know "Ctrl + W, S" will do the same job, we can use that from now on.

How about if I want to change the font size in the code editor?

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-2.png "Embrace Your Keyboard")

Granted, this doesn't have a keyboard mapping, but I can easily select that (with the arrow and enter keys, not the mouse!) and get right to that in the Options dialog:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-3.png "Embrace Your Keyboard")

With a quick "Alt + S", I can adjust the font size as needed.

Adding files to projects, building and publishing solutions, formatting documents  ...  you can find it all with "Ctrl + Q".

## Adding Custom Mappings

So now you know how you can quickly find out how to invoke features in VS. But what happens when a keyboard mapping doesn't exist for something that you use a lot, and you don't have time to keep doing the "Ctrl + Q" + search term dance? A prime example is Test Explorer, which doesn't have a default mapping. Sure, I can find that with "Ctrl + Q". But I want to do three keystrokes, not 9 (on my machine, I have to type "test ex" before that option shows up at the top).

To add a mapping, "Ctrl + Q" and type "Keyboard". This will bring you to the keyboard settings:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-4.png "Embrace Your Keyboard")

This UI isn't the nicest one to deal with, but let's see how we can add a mapping for Test Explorer. First, type "Test" in the Show commands containing: text box:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-5.png "Embrace Your Keyboard")

You should see "TestExplorer.ShowTestExplorer" in the list of results. Select that one, and then make sure the Press shortcut keys text box has the focus. Once it does, use whatever keystrokes you want to map to that command:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-6.png "Embrace Your Keyboard")

I chose "Ctrl + R, S" because "Ctrl + R" is already used by some of the test run commands (e.g. "Ctrl + R, A" to run all tests in the solution), and my fingers are used to "Ctrl + R" being associated with tests in VS. Note that if you select a mapping that is already in use, you'll see it show up in the Shortcut currently used by: drop down list:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-7.png "Embrace Your Keyboard")

With my new keyboard mapping in place, I can easily get the Test Explorer window to show up.

## Exporting Settings

If you end up creating a lot of customized mappings, you may want to reuse them in future versions of VS. It's pretty easy to save them to disk. Use your handle "Ctrl + Q", and type "export":

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-8.png "Embrace Your Keyboard")

Once you select that search result you should see this window:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-9.png "Embrace Your Keyboard")

When you press Next, you'll get a tree view where you can select which settings you want to export:

![Embrace Your Keyboard](https://jasonbock.net/images/Embrace-Keyboard-10.png "Embrace Your Keyboard")

One you get past this screen you can save a .vssettings file. You can use that file to import your customized settings into another instance of VS.

## Conclusion

It may seem a bit silly to devote so much time on this subject. But I can attest that I can get my work done quicker by keeping my hands on the keyboard as much as possible. It's not just VS either; other programs that I use (like what I'm using to write this article, Word), I'm learning their shortcuts as well (and as a side note, having that "Ctrl + Q" feature should be a standard item in any application). The sooner we can get our work out of our hands and into the user's, the better.

Happy coding!

> Published: 11.05.2015 11:20:00 AM CST