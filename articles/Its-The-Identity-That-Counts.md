---
title: It's The Identity That Counts
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# It's The Identity That Counts

## What is Wrong With This Code?

Language: SQL (Oracle) 

```sql
DECLARE
  countServerID INTEGER := 0;
  --  Other declares deleted for brevity.
BEGIN
  select count(java_server_class_id) into countServerID 
    from java_server_table
    where java_server_class_name = 'com.SomeClassName';
    
  IF(countServerID > 0) THEN
    delete from java_class_relation_table 
    where java_server_class_id = countServerID;
    --  Other delete statements deleted for brevity.
  END IF;
END;
```

This script was meant to set up tables in a database that handled mappings between Java classes. It would first clear out any current entries related to an ID if any existed, and then it would add the data. There's more SQL than what I'm showing, but this is all you need to see. Why? Take a close look at the SQL code. If you've done a lot of SQL coding, you've probably already spotted the error, but even if you haven't, you might be able to figure out what's going wrong. 

The first select statement retrieves the number of records that exist in the `java_server_table` table. This is a quick check to make sure that there are records in the table for particular Java class before we start to issue a bunch of delete statements. The problem lies in what is done with that count value. Let's say we find 1 record that matches the criteria we're looking for. Then, `countServerID` equals 1. In this case, the if statement is `true`, and the delete statement is issued. But what if the ID value for `com.SomeClassName` isn't 1? 

Whoops! 

## Possible Solution

The fix is pretty easy. Simply get the ID for the given `java_server_class_name`, and delete records with that ID: 

```sql
DECLARE
  countServerID INTEGER := 0;
  intID INTEGER := 0;
  --  Other declares deleted for brevity.
BEGIN
  select count(java_server_class_id) into countServerID 
  from java_server_table
  where java_server_class_name = 'com.SomeClassName'
  and company_id = 1;
    
  IF(countServerID > 0) THEN
    select java_server_class_id into intID 
	from java_server_table 
	where java_server_class_name = 'com.SomeClassName';
        
    delete from java_class_relation_table 
	where java_server_class_id = intID;
  END IF;
END;
```

> Note: One may wonder why the coder is getting the count value in the first place; why doesn't he just find the ID? The problem was in the way the SQL statement returned the ID value if the class name wasn't in the table - it would return `NULL`. Now, this makes sense, but the problem is a test for `NULL` in an `if` statement didn't work for some reason. That's why a count was retrieved first as that will always return a numeric value. I'll admit - I may be wrong on this, but I couldn't get it to work by retrieving the ID first, and I had to give the developer something to work off of as a template to get these SQL scripts created. I'm all ears if an Oracle guru can get this to work by retrieving the count value. And no, the code I originally gave him didn't have this error in it!  

## It's Not Just The Error, It's The Attitude

Granted, this is a simple problem to solve. What bothered me was that the developer claimed that the code worked once, but then it didn't, and he couldn't figure out why. Can you guess why? If `countServerID` was equal to the desired ID, it worked just fine. And when we only had one entry on the `java_server_table` table (as we did when we were doing preliminary testing), the only ID around was 1. So `countServerID` would always be 1, and everything worked. Once we added other Java classes, the script failed. 

So the developer was right - it would only work under certain conditions. And it took me a couple of times to read the code before I caught it, so I don't necessarily blame him for missing it. The problem is that he didn't take the time to debug the script himself and see why it was failing. If he would've traced the actual SQL statements, he would've immediately seen that the delete statement had an invalid ID value. Remember, try debugging the code first before you ask for someone else's help. You end up learning something about debugging, and you don't waste someone else's time. 

An even bigger problem was the developer's attitude. If something went wrong with his SQL scripts, he never debugged it; he'd just come over by my desk and announced that the code didn't work. I told him about ways that one could debug SQL in Oracle, but he never used the techniques. Plus, he didn't do tasks that I asked him to do. For example, all of the scripts needed to have a company ID to differentiate records between different companies (I didn't show this in the code snippet above because it wasn't necessary given the other problem at hand). So I asked him to add this to all of his SQL statements. Whenever he said his scripts were done, I would check for that addition. After he didn't do it for the third time, I was pretty livid. It wasn't a hard change to make. I reminded him each time to add it. Yet he didn't. 

I don't think I've ever flat-out yelled at someone. I may have been involved in heated discussions with other developers, but getting into shouting matches where you get pissed off about variable naming standards is a waste of time. I kept my cool with this guy, but he was pushing my "just-try-to-get-me-to-work" button. The change wouldn't take more than an hour or two to add. After the third time, I finally told him directly (i.e. face-to-face and not through e-mail) he had to add the company ID to all SQL statements. I don't know what kind of expression I gave him, but it worked. The next time the code was checked in, the statements were correct. 

I eventually e-mailed a good friend of mine about this incident, and I asked him how he would handle it. He came back with some good (and to-the-point) advice: 

"Jason, sorry to hear about this. Seriously, I would set up a meeting with the person and your project manager. I would come to the meeting with a set of reasons that would explain the delays and ask the person to pick one in front of the manager. For example:

1. I didn't understand the request from you
2. I didn't get to your request because of higher priorities
3. I don't actually know how to implement your request
4. Etc.

I actually did this on another project with a developer and as expected, he could only pick the fourth one, which basically equates to, 'I've been fucking off.' You would be amazed at how fast he got his shit together after the meeting." 

If you can't get someone to be productive, then this direct, blunt approach may be the best way to do it. The other three are valid reasons and can be cleared up relatively quickly (so long as they don't get abused and become excuses). But if the fourth one is the answer, then they know the spotlight's on them, and they'd better stop doing whatever it is they're doing (or not doing) and get focused. 

## Summary

1. Make sure you know what your variables are being used for. Sometimes, you may accidentally use them in a way that leads to unexpected behavior.
2. Get a second opinion, as it's always benenficial to get a fresh perspective on a problem. But take some time to debug the problem on your own first before you seek help.
3. Bad code can come from a bad attitude. If you're working with someone who doesn't take responsibility for his/her task list, take action to resolve it. That doesn't mean "fire them," but it does mean "get them back on track fast".

> Published: 04.18.2001