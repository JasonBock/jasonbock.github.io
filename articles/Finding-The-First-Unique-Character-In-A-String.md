---
title: Finding the First Unique Character in a String
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Finding the First Unique Character in a String

Last week I noticed my manager interviewing a potential FTE in a conference room by our cubes. The room has some glass walls so it's easy to see what was going on. Anyway, I saw that he had the candidate writing code on the board to solve a problem. Personally, I generally don't do that in the interviews I run (in fact I don't think I've ever asked someone to do that) but I don't see it as being "wrong" either. I was curious, though, as to the nature of the problem that my manager used, so after the interview was done I asked him what it was. He then proceeded to have me do it on a whiteboard, which, of course, I got it wrong on the first try, but I showed good principles in solving the problem so I think my contract is still intact.

Here's the problem. Given a string, tell me what the first unique character is. For example, if I give you "aAbBABac", you should tell me that "b" is the first unique character. "c" is also unique, but it's not the first one. If I give you "AAAAABBBBB", you would tell me that there are no unique characters in that string.

Got it? Well, my first attempt tried to use collections so I would know if the current character I was looking at was already found. He told me that a candidate he interviewed gave him a solution that does not require any other collections - it looks something like this:

```c#
int index = -1;

for(int i = 0; i < value.Length; i++)
{
  char currentChar = value[i];
  bool charIsUnique = true;

  for(int j = 0; j < value.Length; j++)
  {
    if(i != j && currentChar == value[j])
    {
      charIsUnique = false;
      break;
    }
  }

  if(charIsUnique == true)
  {
    index = i;
    break;
  }
}

return index;
```

You iterate through the entire string, character by character. For each character, you go through the string (ignoring the character itself) and see if it exists somewhere else. If it doesn't, you've found the first unique character. Granted, this code snippet isn't checking to see if the given string is null or is empty. My code has those checks in it; I'm just not showing everything in this example. Also, I'm not returning the unique character; I'm giving the index where that first unique character exists. Po-ta-to, po-ta-to ;).

Here's another approach. It uses a temporary collection to see if you've already found the current character you're searching for. It also uses `IndexOf()` rather than a for loop to traverse the string:

```c#
int index = -1;
int currentIndex = 1;

ArrayList foundCharacters = new ArrayList();

foreach(char currentChar in value)
{
  if(foundCharacters.Contains(currentChar) == false)
  {
    if(value.IndexOf(currentChar, currentIndex) == -1)
    {
      index = currentIndex - 1;
      break;
    }
    else
    {
      foundCharacters.Add(currentChar);
    }
  }

  currentIndex++;
}

return index;
```

I haven't done any performance testing or memory analysis on either approach, although my unit tests all succeed for each solution. I'm guessing the 2nd approach would work "better" if you had a string with a lot of repeated characters and the unique character was somewhere at the end, like "AAAAAAAAAAAAAAAAAAAAb". If a string has a number of repeated characters then the collection will fill up a bit. Maybe Cory will jump in and see which one works better - he likes to do that kind of thing :).

I'm sure these aren't the only ways to solve the problem, and I'm also sure I haven't found the "best" one either. Anybody have any other ideas? Believe me, I'm not fishing for a solution for a problem I'm trying to solve on my current application :). I'm just curious, that's all!

> Published: 07.27.2005 12:32:10 PM CST