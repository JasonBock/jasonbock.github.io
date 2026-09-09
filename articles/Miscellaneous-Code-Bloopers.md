---
title: Miscellaneous Code Bloopers
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Miscellaneous Code Bloopers

## A Small Divergence

So far, I have enjoyed my experience as an angryCoder columnist. I hope that my articles have been beneficial to you. I have learned a lot just by writing them, and the reader feedback has been insightful. With this article, I decided to break the mold that I have created for my articles and list a bunch of small code snippets that I have seen in my career. Some of them made me laugh. Others made me cry. They didn't warrant entire articles on their own, but together they show just how brain-dead we can get at times. It's not that they caused the system to crash; In fact, some are just comments. But when you examine the code you'll understand why developers tend to be very sarcastic. 

Remember, I didn't make these snippets up. I may have altered them a bit, but the essence of what was done lives on in this article. 

## Snippet #1

Language: C++ char 

```c++
someArray[6+1];
```

This one left me rolling on the floor. Who in their right mind is going to declare an array like this? Why not this: 

```c++
char someArray[7];
```

Or even better, use a `#define`. It makes maintenance so much easier as you'll eliminate a magic number popping up in your code: 

```c++
#define ARRAY_SIZE 7
char someArray[ARRAY_SIZE];
```

## Snippet #2

Language: Java 

```java
int data[20];
```

OK, we've covered the array allocation thing already; Get rid of the magic "20" number. My real beef is with the name of the variable. "data". Try to find all occurences of this variable in code using a simple search routine. How many times does the word "data" show up in a comment section? Even if your IDE's search routine is smart enough to find the occurrences of the variable itself, do you have any idea what this variable is used for? 

The name "data" has no semantic value whatsoever. Use descriptive variable names to give future developers a snowball's chance in hell of understanding its purpose for existence: 

```java
public static final int golfTournaments = 20;
int golfScores[golfTournaments];
```

## Snippet #3

Language: C++ 

```c++
BOOL CFile::InsertRecord(void * pStruct)
{
  return TRUE; // the hell with it
  //  more code that'll never get executed.
}
```

Horsewhipping should be a possibility after a code review. This person's head would've been on the chopping block if I saw this. The first statement immediately returns execution back to the client. But there's a bunch of code in `InsertRecord()` after the `return` statement. It's completely worthless. It'll never be run. But every time the project is compiled, in ends up in the executable. 

This is laziness in the extreme. The developer even admits it with his/her "hell-with-it" comment. I would search through the code base and determine when `InsertRecord()` is invoked to determine if I could dump the method entirely. If I couldn't finish the refactoring, I'd at least get rid of the rest of the code in `InsertRecord()`. Even if a compiler was smart enough not to add that code, it shouldn't even be in the code base. 

## Snippet #4

Language: VB 

```vb
'  JohnSmith:  I took out the following line of code.
'  x = x + 1
```

What John is trying to do here is perform source control via comments. Every time a change to the code base is done, it's commented out, and the developer leaves a comment describing why s/he made the change. 

If you're going to reinvent the wheel and roll your own source control tool, for crying out loud, do it right! Don't use comments to achieve source control. A good source control tool will show you what changed from version to version. 

## Snippet #5

Language: Java 

```java
String SavedID;
SavedID = "R"; // "R" stands for "Request for P.O. Copy"
```

I wish I had five minutes alone with developers sometimes. Magic values are bad. But using a magic value and then adding a comment to describe what the magic value means is insane. And it's so damn easy to fix it: 

```java
public static final String REQUEST_FOR_PO_COPY = "R";
String SavedID = REQUEST_FOR_PO_COPY;
```

## Snippet #6

Language: VB 

```vb
Dim bolTest As Boolean

bolTest = optRefreshTestData.Value
oTestData.Update bolTest
```

This is one my own "gems." I created a useless `Boolean` to pass into an object's method, and I never use it again. However, the option button's `Value` already has the desired true/false value in it, so why create another variable? Just get rid of `bolTest`: 

```vb
oTestData.Update optRefreshTestData.Value
```

## Snippet #7

Language: C++ 

```c++
void someMethod()
{

}/* end of function */
```

Well, duh. I don't need to be told that the curly brace is the end of the method with a comment. This is completely unneccessary. Never use comments to describe the obvious, like this: 

```c++
//  Increment i
i++;
```

## Final Comments

I'm sure you've seen your fair share of "one-line" wonders in your career. Either you wrote them yourself or you found them in someone else's code. Eventually, you run across a shining example of what we're really producing in our systems. They're small, they compile, and they don't hurt the system; But the lack of intelligence behind the code is evident. Ultimately, there's not much you can do other than make the slight improvement and move on. Keep in mind that code bases can last a very, very long time. Make sure that each line of code is as solid as you can make it. 

> Published: 06.08.2001