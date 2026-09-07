---
title: `IHttpModule` is Cool, A Weird ASP.NET Compiler Error, and Good Luck, Marquette!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `IHttpModule` is Cool, A Weird ASP.NET Compiler Error, and Good Luck, Marquette!

Today, one day before the release date, we ran into a couple of unexpected problems. One is that the production server uses a web farm, which screws up how how state is being managed. Granted, ASP.NET has a fair amount of options to get around this, but we didn't expect this configuration, and, more to the point, we didn't expect that the admins wouldn't set up the configuration file correctly. The biggest problem, though, was how we'd handle the web site URLs. See, the site is set up so that you can access it through two URLs, which I'll call www.fakesite.com and www.realsite.com for illustrative purposes. What we thought would happen is that any request to www.fakesite.com would be automatically redirected to www.realsite.com. However, if a user came to the site via www.fakesite.com, we needed to change the look of the site on a couple of page. (Don't ask - I didn't design the site.)

Well, everything worked as expected during test, but in production, they decided to map the two URLs to the same IP. Without getting into the gory details, this screwed up our logic big-time, and the owners of the project (basically a marketing firm that gets **really** upset if an image is off by 1 pixel or just a tad wrong in the color) did not like that our logic didn't work. Fortunately, as I listened to the lead talk on the phone and picked up what the problem was, I remembered that trusty interface called `IHttpModule`, and I was able to whip up a solution to intercept the HTTP request and handle redirections plus our original logic correctly.

The `Uri` class also made the parsing of the incoming request dirt simple. I remember Craig Andrea showing the simple yet time-saving `Path.Combine()` to handle combining directories and files without having to worry where that pesky slash is ("at the end of the directory?", "at the beginning of the file?" "Neither?" Doesn't matter with that method!). Well, the `Uri` class makes combining the parts of a web request very easy. The best thing about all of this is that we can back out my change if something goes terribly wrong with a simple change to the configuration file. We have a backup idea in case my HTTP module doesn't work, so it's no big deal to fall back to plan B if needed. God, I love the configuration possibilities that .NET has to offer! Tomorrow will be the real test, but at least I was able to offer up a solution that should take care of the issue, and it really didn't take that long to cruft up. .NET - sometimes it's a beautiful world.

We were able to solve another problem that's been bugging the hell out of me since I found it. We need to set the background image on the `<body>` depending on what URL the request came from (yeah, it's done because of the two URL issue I mentioned before). Anyway, I thought, "well, make the `<body>` element runat="server" and change the background attibute in the code-behind." But when I did this, I got a rude shock when I saw our error page pop up. A quick check to the Event Log produced this information:

```
Error Type: System.Web.HttpCompileException
Error Message: External component has thrown an exception.
Error Source: System.Web
Error Target Site: Void ThrowIfCompilerErrors(System.CodeDom.Compiler.CompilerResults, 
   System.CodeDom.Compiler.CodeDomProvider, System.CodeDom.CodeCompileUnit, 
   System.String, System.String)
Stack Trace:    
   at System.Web.Compilation.BaseCompiler.ThrowIfCompilerErrors(
      CompilerResults results, CodeDomProvider codeProvider, CodeCompileUnit sourceData, 
      String sourceFile, String sourceString)
   at System.Web.Compilation.BaseCompiler.GetCompiledType()
   at System.Web.UI.PageParser.CompileIntoType()
   at System.Web.UI.TemplateParser.GetParserCacheItemThroughCompilation()
```

Uh ... what?

OK, I get that the CodeDOM compiler is dying for some reason or another. And I know that the HTML isn't the cleanest (one of the things I've learned on this project is to validate **everything** - the next project I'm on will make sure we have valid HTML, that's for damn sure!), so the ASP.NET compiler may be barfing on something in our layout. But I have no idea where to start. I tried to run our HTML through a validator and I tried to change some stuff, but nothing worked. Hopefully a future version of the ASP.NET compiler will yield more helpful information than this (and maybe it does and I don't know where to find it), but that doesn't help me now. So I ended up emitting some JavaScript on the fly to change the background during the onLoad event of the <body> element.

Well, this worked, but we ended up with a jerky look to the page, because the original background image URL got loaded first, and then our new one took its' place, which did not lead to a wonderful experience. Since the testers haven't seen the site with this altered look when it comes from www.fakesite.com, we didn't see a bug, but I knew it was just an inevitability. I talked to the lead on the project, and because he hadn't had his nose to the grindstone on this problem, he was able to have a fresh approach to it and suggested that I used a `<div>` element **inside** the `<body>` element, and control its background style in the code-behind. And, guess what, that worked perfectly! I have to admit that I like to work alone when I'm tackling a problem, but it's always nice to have other views as they may catch things that I've completely missed that are so freakin' obvious.

Finally, my old stomping grounds - Marquette University - has its' men's basketball team in the Final Four. Wahoo!! Of course, I'm sure some alumni remember the days when the team was called the Marquette Warriors (and I know some people who absolutely refuse to call them the Golden Eagles, but, hell, it's better than one of the other suggested name: Lightning. God, that sucked). I admit, I didn't follow the team closely when I was in school, but at least they're in and my wife's team (the Wisconsin Badgers) didn't make it past the Sweet Sixteen. My wife and I have some good-natured digs about each other's respective alma maters, but this time I have one up on her.

> Published: 03.31.2003 11:00:58 PM CST