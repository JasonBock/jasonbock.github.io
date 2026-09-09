---
title: Wheels Within Wheels In A Spiral Array
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Wheels Within Wheels In A Spiral Array

## What is Wrong With This Code?

Language: InstallScript 

```
function DoesApplicationExist(ApplicationID)
    BOOL bExists;  
    NUMBER i;
    OBJECT oCOMAdmin; // as COMAdminCatalog
    OBJECT oCOMApps; // as COMAdminCatalogCollection 
    OBJECT oCOMApp; // as COMAdminCatalogObject 
begin
    bExists = FALSE;
    set oCOMAdmin = CreateObject("COMAdmin.COMAdminCatalog");

    ShutDownApplication(ApplicationID);
    set oCOMApps = oCOMAdmin.GetCollection("Applications"); 
    oCOMApps.Populate();   

    for i = 0 to (oCOMApps.Count - 1)
        set oCOMApp = oCOMApps.Item(i);
        if (oCOMApp.Key = ApplicationID) then 
            bExists = TRUE;
        endif;
    endfor; 

    if (bExists = TRUE) then
        return 1;
    else
        return 0;
    endif;
end;

function ShutDownApplication(ApplicationID) 
    OBJECT oCOMAdmin; // as COMAdminCatalog
begin   
    if (DoesApplicationExist(ApplicationID) != 0) then
        set oCOMAdmin = CreateObject("COMAdmin.COMAdminCatalog");
        oCOMAdmin.ShutdownApplication(ApplicationID);
    endif;
end;
```

These two functions were used in an InstallShield project I was working on. Since InstallShield doesn't handle COM+ application installations, I needed to use the COM+ Administration objects so that I could create applications, add components and roles, etc ... I wrote a bunch of utility methods in a script file that generalized some of the tasks that I needed, but as you'll see in a moment there's a big problem with the code. 

`ShutDownApplication()` is used by most of the other COM+ utility methods to ensure a COM+ application is shutdown before tasks like adding components are attempted. However, I also wanted to make sure that the application existed before I tried to actually shut it down. There's no simple way to test for an application's existence using the `COMAdminCatalogCollection` object; You have to get a collection of every application and search through the list (at least I couldn't find a simple way to do this). So I wrote a `DoesApplicationExist()` method. 

The problem comes in `DoesApplicationExist()`. For some odd reason, I put a call to `ShutDownApplication()` from `DoesApplicationExist()`. I'm sure you can guess what happened. When I created the setup EXE and ran it, it chewed up all of my memory. Of course, I immediately assumed it was InstallShield's fault with their ikernel.exe file, as that was the one that became the memory hog. Let me explain why I came to this conclusion. Before I created the "real" setup EXE, I used InstallShield to create a prototype EXE. Because I wasn't too familiar with InstallShield, I wanted to make sure that I knew what I was doing before I tried to create setup EXEs for our production components. I eventually figured out what InstallShield could and couldn't do, and I got my prototype to do what I wanted it to. Since the production setup EXE was similar in functionality to the prototype, I assumed that there was something wrong with InstallShield. This line of thinking is what got me going down the wrong path. 

I started to post messages in the InstallShield newsgroups and I asked around the office if anyone had ever seen problems like this. And when I searched the newsgroups for posts related to ikernel.exe, I saw messages that hinted that ikernel.exe wasn't very stable. Unfortunately, this information kept me down the wrong path - that is, it's InstallShield's fault, not mine. 

> Note: For the insanely curious, you can find my post by searching for "Jason Bock ikernel" at the Usenet archive. As you can tell by my post, I did switch something else in the setup configuration, and that also clouded my debugging judgement.  

Fortunately, I ran into this problem near the end of the work day, so I went home and forgot about it. The next day, I was in a meeting and we were talking about out project statuses. I mentioned my problem and the approaches I was going to take to solve it - i.e. trim down the installation, call other InstallShield users, call InstallShield for tech support, etc ... But for some reason, my brain told me that I should review my code - maybe something I did was the culprit. And then it dawned on me that maybe I was recursively calling a function; That would explain the wild behavior of the setup EXE. 

As soon as I got back to my desk, I saw the problem. By calling `ShutDownApplication()`, I call `DoesApplicationExist()`, which calls `ShutDownApplication()`, which calls ... well, you get the picture. No wonder the program took up so much memory! This is recursion at its worst, as there are no checks to stop the functions from calling each other. Granted, I didn't mean to have recursion; But it happened, and it wasn't pretty. 

## Possible Solution

Simple. Get rid of the call to `ShutDownApplication()` in `DoesApplicationExist()`: 

```
function DoesApplicationExist(ApplicationID)
    BOOL bExists;  
    NUMBER i;
    OBJECT oCOMAdmin; // as COMAdminCatalog
    OBJECT oCOMApps; // as COMAdminCatalogCollection 
    OBJECT oCOMApp; // as COMAdminCatalogObject 
begin
    bExists = FALSE;
    set oCOMAdmin = CreateObject("COMAdmin.COMAdminCatalog");

    set oCOMApps = oCOMAdmin.GetCollection("Applications"); 
    oCOMApps.Populate();   

    for i = 0 to (oCOMApps.Count - 1)
        set oCOMApp = oCOMApps.Item(i);
        if (oCOMApp.Key = ApplicationID) then 
            bExists = TRUE;
        endif;
    endfor; 

    if (bExists = TRUE) then
        return 1;
    else
        return 0;
    endif;
end;
```

There's no reason to shut down a particular COM+ application just to see if it exists. 

Admittedly, I felt like an idiot when I found this. I made a wrong assumption about where the problem was, and it took some time to realize that it wasn't InstallShield's fault at all. At the same time, by letting the problem go for the evening, I gave my brain a rest. I've noticed in my career that whenever I do this, the problem eventually works itself out even though I'm not thinking about it. This time it prevented me from expending even more effort going down a path that simply wasn't the correct one, saving time and money (and it also helped me save face). 

## Summary

1. Don't assume it's the program's fault. You may be doing something that's causing the problem.
2. Get away from the problem for a while. It's amazing what the mind will do for you if you stop working on a problem.

> Published: 05.04.2001