---
title: Transferring and Redirecting With Passport
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Transferring and Redirecting With Passport

I ran into an interesting scenario today. Up to this point, I've been using `Response.Redirect()` to move from one page to another because I couldn't find the method to do a page forward on the server. I finally found it today (I knew it had to be there somewhere!) via `Server.Transfer()`. OK, I changed the code accordingly and off I went.

Well, it wasn't as easy as that. There's one section of the site (we'll call it Initial User Entry, or IUE, for now) where we need to do Passport authentication - that is, the user needs to be signed in to access this section of the site. Before today, it was working as expected - the web.config file was set up to make sure the IUE pages could only be accessed by those who were signed in. Unfortunately, when I changed everything over to `Transfer()` calls, the IUE section didn't work as expected. The first page came back as though I was already logged in; it should've done a redirect to a login page so the user can sign into Passport.

The problem was when the main page transferred the request to the first IUE page, the Passport module is already out of the picture (at least I think this is what's going on). It's determined that the main page doesn't need Passport authentication, and therefore doesn't check the transfer to the first IUE page. But by changing the main page to redirect to the first IUE page, the Passport stuff kicks in as expected. This makes (some) sense as the redirect tells the client to go to the first IUE page, so the Passport authentication module will pick up the request to the secured page.

I'm not sure how many people are using Passport, and I may be wrong with my analysis. But just make sure that if you use Passport in your ASP.NET application and you're moving around from page to page, be careful when you're using `Transfer()` over `Redirect()`.

> Published: 02.05.2003 05:57:52 PM CST