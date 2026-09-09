---
title: Suggestions for VB7 and Beyond
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Suggestions for VB7 and Beyond

## Introduction

As each version of Visual Basic rolls on by, many people voice their opinions about the product. Some rave about the ease of creating Windows applications, some wish for more programming power (and continue to do so as each release does not fulfill their requests), and some ... O.K., more than some complain about version conflicts, service pack upon service pack (that take forever and a day to download over a modem), constricting development environments, at least 5 different ways to access data, etc., etc. One very prominent member of the VB development community, Bruce McKinney, finally gave up with the tool and refuses to talk about it anymore.

So what can be done in future releases? Personally, I've been kind of miffed with the tool since VB6 was released. I liked VB5 - to me, it was a much better release than VB4. However, VB6 appears to have gone backwards and is more geared towards business developers. That's fine in some ways, but I think there's areas where the tool can go through some significant improvements. So here's my list (in no particular order of importance and/or complexity) of what I think should be done with the tool in the months and years to come.

### 1: Add More Fundamental Operations

If you've programmed in C++ or Java, you've probably used the following syntax to increment and decrement a value:

```
i++;
i--;
```

It's much cleaner (and faster) than doing this:

```
i = i + 1;
i = i - 1;
```

So why doesn't VB have an equivalent? I have no idea. I run into these operations all the time in VB, and it's really annoying to do this:

```vb
lngCustomerCount = lngCustomerCount + 1
```

when I'd like to be able to do something like this:

```vb
lngCustomerCount++
' Or...
Incr lngCustomerCount
```

PowerBASIC, another BASIC compiler, has the `Incr` and `Decr` functions to perform these tasks. Implementing them into the language wouldn't be that difficult; I'm still amazed that something like this still isn't in VB.

Another set of operations are bit shifts. Granted, you're probably not going to run into them very often in VB, but it would be nice to have them around. For example, one way to multiply a number by 2 is to do this:

```vb
lngNumber = lngNumber * 2
```

But if VB would add a bit shift left operator, something Javaish like this:

```vb
lngNumber<<1
```

the resulting operation is faster. Bit-shifting operations are not that hard to implement, they do pop up enough to make themselves useful (depending upon what you're doing) and I wish VB would add them.

### 2: Interface Definitions

After spending some time using Java, I found out that I like the language. One thing that's really nice is being able to define an interface using the `interface` keyword, like this:

```java
public interface MyInterface
{
  void MyCall();
}
```

However, using VB to create interfaces is kind of annoying. I've found that it's far easier to use IDL to compile a type library than using a ActiveX DLL project just to define an interface definition (I won't go into the details here, but it's just plain awkward). What would be cleaner is to add another property to the class module that would let you choose between a `Class` or an `Interface`. `Interfaces` could contain enumerations and interface definitions only; no implementations of the properties and methods would be allowed.

After spending some time using IDL and creating type libraries, I've found that I really like keeping interfaces and implementations separate. Right now, you really can't do that from within the VB environment. It would be really cool if VB added this.

### 3: Implementation Inheritence

Uh, oh. The argument that never gets anywhere within the object development community. First of all, I'll state that I'd like to see it added to VB. Sure, I might shoot myself in the foot with it and run into the "fragile base class" problem. So what? Anything misapplied can wreak havoc in a system, so why is this any different? I can easily make a runaway recursion routine in VB - is that a good reason to get rid of recursion? There are cases where implementation inheritence would be really useful, and it stinks when you don't have it.

But it's not as simple as that. VB objects are COM objects, period. And since you can't do implementation inheritence in COM, you can't do it in VB. Therefore, we either have to wait until:

* COM supports implementation inheritence (and there's good reasons why that might not ever happen - see #28 in "Effective COM" for an good explanation of this).
* VB redesigns how classes are done, similar to C++ or Java, where you can inherit until the cows come home. But once COM comes into the picture, you're done.

My guess is that VB will never support implementation inheritence. COM+ won't have it (at least from what I know at the time of writing this article), and I doubt that the VB team is going to rip apart how classes work in VB simply to support implementation inheritence. But I'll still ask for it.

### 4: Native Threading

I'll keep this simple: make calling `CreateThread` very simple to use in VB7 or add a function that makes it painless to spawn new threads. Sometimes, I don't want to make a COM EXE simply to add another thread into the picture. Again, look at PowerBASIC. It has a bunch of Thread functions to make threading easy to use. I don't argue that multithreaded applications can get tricky and that they can get misapplied, but, again, if I need it should be easy to use. I applaud what Matt Curland wrote in the Black-Belt column of the June 1999 issue of VBPJ. But I really don't want to add that code into every threaded function I need.

### 5: No More AutoIDispatch, Please!

This one's kind of obscure. Whenever you create a class in VB, VB automatically creates a hidden interface for your class (this needs to be done via the rules of COM). This interface has an underscore added in front of the class name you give. For example, if you had a class called `Customer`, VB creates an interface called `_Customer` for you to fulfill COM's requirement. Furthermore, VB will implement the `IDispatch` interface for your for each class you make, since that hidden interface derives from `IDispatch`. For the most part, this isn't that big of a deal and it automatically makes your objects accessible to scripting clients, but there's no way to turn this off.

I'm no ATL whiz (hell, I can barely read C++ for that matter), but I have messed around with it, mostly with the ATL Wizards (they make my life easy). There's one screen that caught my eye the first time I saw it:

![VB7 Fixes](https://jasonbock.net/images/VB7Fixes.png "VB7 Fixes")

Notice the *Interface* section? You can choose between custom and dual interfaces. All this means is that, if you choose custom, you interfaces don't inherit from `IDispatch`; they inherit directly from `IUnknown`.

Now why can't we do this in VB? I realize the VB is building a big market around MTS and using VB to create those middle-tier components. Of course, ASP fits into that picture as well. Which means that these objects need to be accessed from scripting language. Hence, the need for `IDispatch`. It's far easier to simply make a dual interface and make the object accessible anywhere than letting a VB programmer choose otherwise and possibly shooting a big hole in their feet when ASP developers can't use the object. But I should have the option to choose either way. If I don't want it, I shouldn't be forced into having it.

### 6: Pointers (wait, what did I say? ... )

OK, I'll admit, this isn't a big deal. If they were never added, I'd live a very happy VB programming life. But the odd thing is that VB is already half-way there. The `VarPtr`, `StrPtr`, `ObjPtr`, and `AddressOf` functions are there in VB (`AddressOf` being the only one supported, though). So while you can get the pointer to virtually anything in VB, you can't dereference it. In a nutshell, this stinks. There are times where these functions come in very handy, but I feel cheated in a way. Again, PowerBASIC has the support for pointers. Why not VB, if they're already giving us one side of the story?

I won't hold my breath on this one, and quite frankly I don't need it. I just find VB in a weird state with this one.

### 7: Subclassing

With the addition of the `AddressOf` keyword in VB5, developers were able to call more Win32 APIs than before. Some of those were ones that allowed you to subclass a window, which opened up new possibilities for window management. Problem is, it's a pain in the butt. Which is precisely why virtually every personal and commercial VB web site around has a subclassing control.

I'm all for innovation. But it would be nice if VB added a control that you could plop on a form and subclass away. And just because VB would have a control that does subclassing doesn't mean that those companies that make subclassing controls would suddenly go under (Sheridan and Desaware are still going on even though a lot of what they provide could be done within VB; they just make it easier). It's a small thing and if it's not there it's no big whoop.

### 8: Better VB Component Design

I'm no OO guru, but I have spent some time studying it. One thing that keeps on coming out is that you should try to make your definitions clean and small. A class with hundred of methods is harder for a developer to use than one with 5 or 7. This isn't a hardcore rule that must be followed at all costs, but it's a good one to keep in mind.

So let's take a look at VB's run-time DLL, shall we? The first thing you notice is that it weighs in at 1.4 meg. In this day of 8 gig hard drives, it's not that big of a deal. But have you ever noticed how many functions are exported in this DLL? There's over 1900 of them. For some projects, I really have to wonder how many of those are used. It would be much more efficient if the run-time DLL was split up to only contain functions that are related. What those would be, I have no clue. VB doesn't document its' run-time DLL, so your guess is as good as mine as to what _Clatan is used for. But if there's no reason to port over a 1.4 meg DLL when I don't use over 50% of the functionality, then that's wasted space.

Another good example of this is the common controls OCX file. This file contains a lot of controls, like the tree view, progress bar, and list view, just to name a few. But look at what the Common Controls Replacement Project has done. Not only do they fully implement the controls (only until VB6 SP3 are VB programmers able to make the progress bar vertical), they're in separate files. If I only want a progress bar why in the world should I have to include all the others?

I guess that having a big honking file makes it easier to upgrade. For example, if I'm using a progress bar and now I want to use a tree view, I only have to update the EXE. By having separate components, I'd have to install another OCX file. But I think having well-defined components is a much more elegant design in the long run. And program bloat is program bloat. If I only need a 30 KB file then there is no reason why I should be forced to have a 300 KB file. I don't care if that's insignificant compared to the hard drive space. It's simply a bad design.

### 9: HTMLHelp - HELP!

This one goes without saying. I like the idea of using HTML for the basis of help files, but something got way out of whack with Microsoft's HTMLHelp. It's God-awful sloooooooow. Whatever needs to be done here, do it quick.

### 10: Dead Code Removal

You'd think that VB6 would have something in the IDE to let you know of dead code - i.e. constants, enumerations, procedures, etc., that aren't being used anywhere in the program. But no, it still gleefully compiles that code into your DLL or EXE, bloating it unnecessarily. Visual C++ will give you a warning when something like this occurs. Why VB doesn't is beyond me.

## Conclusion

There - my VB wish list. I did send this list to Microsoft - you can make your own wishes by sending an e-mail to vbwish@microsoft.com. I realize that some of my suggestions probably go beyond what Microsoft ever intends to do with VB, but at least I know I said my peace. Granted, most VB programs won't ever need some of the things I asked for. But I shouldn't have to hit a brick wall when it wouldn't be that hard to add a door. And from what I've read in VB magazines and newsgroups, I'm not the only one who's asking for some changes. Maybe it's easier just to say that VB isn't where I need to be at times. But when I see what PowerBASIC is doing with their tool, I wish that the VB developers at Microsoft would wake up and see that you can write extremely tight code using the BASIC language.

I'm interested to hear if you have any other suggestions and/or criticism with my ideas - e-mail me if you do. I'm sure I didn't hit all of the issues that other developers have. If it's good fodder I'll add it to this article as an addendum.

## Addendums

### 07.07.1999
emunews@aol.com sent me a list of 10 suggestions that he has for VB7. Some overlap with mine, and some add to my list. Here's the list verbatim - I'll add some comments at the end.

> 1) Inheritance.
> 2) Revised Menu Editor using a treeview metaphore
> 3) Strict Type Checking option. The assignment of variables of different dataypes (a string to an integer, for example) would generate a compile time error instead of run-time bugs. This would do for assigment operations what Option Explicit did for variable declarations.
> 4) Ability to add icons to menus.
> 5) ReDim should not override Option Explicit. Right now, VB will actually let you redim an array that doesn't exist (it quietly creates a new one) even if you use Option Explicit. This should generate a compile-time error, not run-time bugs. (If Microsoft is worried about breaking existing code, this can be an option in settings.)
> 6) A simple, automated way of setting tab stops correctly. TabOrder doesn't cut it.
> 7) Fix the 200+ known bugs in VB6. It's hard to create stable, robust apps when VB6 has too many bugs of its own.
> 8) Revised Err object. New features should include ProcedureName, ModuleName and LineNumber properties, plus a CallStack collection.
> 9) Animated GIF control
> 10) Optional Static Linking. EXE's compiled with this option should only contain the functions actually used by your program. VB's resource hungry, bloated MSVBVMXX.DLL files have always been overkill. Let's make them optional with a compiler option similar to VB5/6's P-Code vs Native Code compiler option.

I completely forgot about the Menu Editor. That definitely needs to change (for example, you shouldn't need to jump into the APIs to add an icon next to a menu item). The `Err` object additions make sense for the most part - they would make finding bugs much easier to do. The only one I'm not sure about is the static linking option. Considering how many bugs VB has before the service packs plug (some) of the holes, I'm not sure I'd **want** to statically link VB's code. For small programs, this makes sense, but it would have to be used wisely.

Here's another idea I came up with today: Variable in scope. While the *Locals* window can tell you everything you need to know about what's currently local to a procedure, what I want is a drop-down list (like the *QuickInfo* list) that tells me each and every variable that currently accessible from the procedure when I'm coding. I'd like to know the variable name, what the variable type is, and its' scope (local, module, or global). My guess is that this would be invoked by hitting something like Ctrl-B or some other combination of keys. I would personally find this to be a nice feature to have (I know that some Java IDEs already have this).

### 07.21.1999

Charles Mauldin sent me 6 suggesstions, 3 of which are new:

> 1) Hi Resolution Timer. The Timer control is inadequate because of it's 55 ms resolution. What is needed is a control which has at least 1 ms resolution. I am currently using CCRP Timer control which is fantastic.
> 2) New Variable Types. All Integers and Longs in Basic are signed, while the Byte type is unsigned. It would make better sense to create three new types of variables, uInteger, uLongs, and sByte.
> 3) Fix the bugs in Dictionary Object. The Dictionary Object is one of the best thing that VB has going for it, if it only worked as the sparse documentation claims. The Dictionary object could be enhanced by adding sort and filter methods.
> 
> I hope this has given the VB Team some insight into what a long time Basic programmer would like to see improved in the next release of VB.

All of them seem fine by me. I especially like the data type one, as a signed `Long` would exactly map to the `DWORD` type that pop up all the time in API calls.

Corrado submitted his list, 4 of which are shown here:

> 1) DIM x as integer=3 or DIM x as long=MyFunction() (By an IDE switch)
> 2) Standard DLL creation, Ok we have ActiveX DLL ... but registering/unregistering is sometimes annoying...
> 3) A real installer like Installshield or Wise (Why IS Express is included in VC but not in VB?)
> 4) Project Explorer should display class hierarchy.

I really like #1. Variable initialization is definitely needed. I also totally forgot about allowing us to export functions in DLL. PowerBASIC makes it brutally easy with the export keyword. The installer issue is not that big of a deal to me. The class hierarchy idea would be nice, but if it's not there I wouldn't miss it.

Brian McDougle sent these two suggestions:

> First, provide a WinMain routine for every form which is called prior to calling the actual WinMain routine. This will eliminate the need to subclass the form just to reply to a message that your code has sent to the form. For example, I have written a number of small programs that parse web page log files. They run a loop reading each line of the file and use DoEvents so that the user (mainly me) can click on a "Stop" button to stop the process if necessary. It would be nice if I could eliminate the loop and have a single routine that reads a line from the file, parses it, and then posts a unique message to the form that tells it to read the next line. The "WinMain" routine would get the message and call the routine to read the next line and then set the appropriate return value to indicate that the message was processed. I realize this can be done with a Timer control but why have the overhead when all I need is a simple mechanism to "prompt" the form to do something.
> 
> Second, it would be nice if pieces of a menu could be copied and pasted onto another form without having to edit the VBP file directly. The Menu Editor should allow the user to select specific menu items and copy them to the clipboard, or (better yet) to an RC file (like you do for C programs). It could them permit the user to paste either from the clipboard or from an RC file. This would simplify the task of creating multi-language applications since only the RC file would need to be sent to an outside source to be translated.

The `WinMain` idea is interesting, although having a subclassing control to make things easier would help out as well. I really like the idea of somehow having the ability to copy menu structures from one form to another. Another To-Do for the Menu Editor updates ;).

David Crowell made two interesting points in his e-mail:

> I really like VB5, and have no intention of using VB6, but I will upgrade to VB7 if they include a way for callbacks to a class or form procedure such as:
> 
> ```vb
> Dim ThisThing as CThing
> Set ThisThing = New CThing
> Call SomeAPIFunc( x,x,x, AddressOf ThisThing.CallBackProc)
> ```
> 
> or within the class (or form):
> 
> ```vb
> Call SomeAPIFunc( x,x,x, AddressOf Me.CallBackProc)
> ```
> 
> That touches on your subject of subclassing, but this is more generic, and doesn't require a control or a form. Imagine how easy it would be to create a timer class, and you wouldn't need a .bas module either.
> 
> Also, what about 'lightweight' classes. They wouldn't use COM, but would have to be private. This could definitely decrease memory usage for apps using many instances of classes.

Using `AddressOf` to call class or form functions is something that other VB developers have asked for. Bruce McKinney really doesn't like this operator because of this limitation. People have said that this can't be done because of vtables in COM, but now that I've looked into the depths of COM, I really don't see why `AddressOf` can't be extended to forms and classes. You end up with a function pointer anyway in the vtable, so what's the big deal? (OK, maybe there is one and I'm just too dumb to see it, so flame away and I'll take the burns if I deserve them.)

And having COM-less classes? Heresy! Seriously, I totally agree with this idea - it relates to the implementation inheritence issue I brought up. By making Visual Basic truly object-orientated by having a notion of classes separate from COM, we could finally have inheritence.

Karch Greg had these ideas:

> First, what I would really like to see would be an integrated resource editor that only compiles the resource file when you run or compile a project, instead of creating one by yourself in Notepad and compiling it every stupid time when you modify only one, one file in it. VB's poor support for resource files drove me to write my own resource file manager for Visual Basic.
> 
> Second, what I would really love would be true stand-alone executables, where the components and calls that you use get compiled directly into the exe or control or dll, like Delphi or, for a surprise, VISUAL BASIC 1.0 for DOS!
> 
> Third, I think a better menu designer is needed, like the one in Visual C++, where your menu items automatically get named when you type in the caption and that you can visually design your menu. You could type in the caption directly, drag and drop the item where ever you want it.
> 
> Fourth, these aren't necessary for everyone, but maybe support for bitmaps in menus, subclassing of any form or control where an event gets fired, eliminating the need for AddressOf, and exported functions would be really nice!

Hey, Microsoft, notice a trend here? A lot of issues are popping up over and over again. The static linking issue made me think about PowerBASIC again. Oh, if we could only have true EXEs and DLLs! And now with their DDT stuff in their new 6.0 release, they are really going to start making VB programmers take notice. (Yeah, they don't have COM yet, but they will .. .someday).

And finally, Christian Schmitz has these comments about VB7:

> Support SetParent for different forms
> 
> Support basic pointer operations and low level inplaced function like memory copying (without them I can't implement any basic data handling efficiently like own linked lists or collections) I won't use more that one language to solve my problems
> 
> A class library "VBC" as a sorted place for the basic functionality (it's much more cleaner than a ever growing collections of functions)
> 
> Optional Static linking inclusive all external components
> 
> Improved VB-Performance
> 
> Improved Editor which allows the grouping of functions (like a treeview and only for view purpose to get a better overview) inside a class or form an so on
> 
> Attachment of windows messages, so that the appear as normal events (type safe parameter without the need of copy the Data via Copymemory)

### 07.27.1999

Mark A. Goodell sent me these very insightful suggestions:

> 1. REAL structured error handling. Most high quality languages these days support exceptions. Something like:
> 
> ```vb
> Try
>   Set cn = New Connection
>   cn.Open "DSN=Northwind"
> OnFail
> If TypeOf Err.Exception = excADO Then
>   MsgBox "An error occurred."
> End If
> Finally
>     Set cn = Nothing
> End Try
> ```
> 
> When teaching Visual Basic I have to routinely apologize for the all the hassles and pain of using the On Error ... statements.
> 
> 2. Object constructors/destructors with arguments.
> 
> 3. Fix the WithEvents implementation to work with control array control references. If you have a variable defined WithEvents:
> 
> ```vb
> Private WithEvents m_txt as TextBox
> Sub Init(txt as TextBox)
> Set m_txt = txt
> End Sub
> ```
> 
> You can not set the variable to a reference to a control in a control array:
> 
> ```vb
> Init Me.txt(2)
> ```
> 
> I would say that fixing the menu designer would be a good idea, but then again maybe they should keep it. It has been a running joke for so long.

I like #1. I forgot all about SEH - I should've known better especially after learning Java. #2 is going to be "fixed" with COM+. I believe that you'll have constructors that you can pass arguments to, but I'm not sure about this. The comment about the menu editor was so good I had to keep it in.

By the way ... here's some more information about the AddressOf thing with class functions. Here's the problem in a nutshell:

Say you wanted the API call `EnumThreadWindows` to callback to one of your objects. Here's the function signature that `EnumThreadWindows` expects for the callback:

```vb
Function EnumThreadWindowsProc(ByVal hwnd As Long, ByVal lParam As Long) As Long
```

No problem, you say. You define this function in a class called `APICallbackProcs` (assume we're working with in-process components here), make an object called `objAPICallbackProcs`, and pull the function address via some real funky calls to `CopyMemory`:

```vb
lngPObj = ObjPtr(objAPICallbackProcs)
CopyMemory lngPtrAddress, ByVal lngPObj, 4
lngFAddress = lngPtrAddress + (8 - 1) * 4
CopyMemory lngDesiredAddress, ByVal lngFAddress, 4
```

Is this Greek or what? Don't worry about it - this code is basically walking the vtable to grab the function address, which is stored in `lngDesiredAddress` (see the Black Belt Programming section of the March 1997 issue of VBPJ). The reason I won't explain the code is that it doesn't help us out anyway.

If you pass `lngDesiredAddress` in to `EnumThreadWindows`, it will try to call `EnumThreadWindowsProc` on `objAPICallbackProcs`. But here's the problem. Even though you thought you defined this correctly in VB, there's some COM magic underneath the scenes. What `EnumThreadWindowsProc` **REALLY** looks like is this:

```vb
Function RealEnumThreadWindowsProc(ByVal This As APICallbackProcs, ByVal hwnd As Long, ByVal lParam As Long, ByVal retVal As Long) As Long
```

Any function in a COM vtable takes as its' first parameter a reference to itself (Sort of, but not really. Well ... just read "The COM Programmer's Cookbook" at http://msdn.microsoft.com/library/techart/msdn_com_co.htm. I don't want to get into the details of this right now!). Furthermore, the return value you get with `RealEnumThreadWindowsProc` is the `HRESULT` success/error COM code. The `retVal` is the return value that you get with `EnumThreadWindowsProc`. VB magically checks the return value and if an error occurred, it raises an error that you can catch. If everything was OK, you get the value of `retVal` as the return value you see.

The bottom line is that what the API call wants to see as the function signature isn't what is there, even though you thought it was. Hence, memory exceptions of the most glorious kind will probably smack you in the face. And I don't know a way around this one, because you'll never be able to change how your class functions are really defined for COM compliance.

The only way this could be eliminated is for VB to have an interception layer when an API call tries to make a callback into a object function. But for now, I don't see a way that it can be done with pure VB code.

Keep the comments coming in, especially if you see something that hasn't been covered yet.

### 12.14.1999

Well, it's time to close this article. I received even more e-mails after the last update, but most were repeating what was already said ... and I lost most of them when I upgraded my PC to NT (d'oh!). Thanks to all of those who voiced their opinion - make sure to send Microsoft your wishes for VB. I don't know if we'll see any of the changes listed here, but I really hope that VB7 is a release for the developers.

> Published: 06.28.1999 12:00:00 AM CST