---
title: Using the Amazon.com Web Services: A Quick Overview
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using the Amazon.com Web Services: A Quick Overview

As I mentioned yesterday, I'm playing around with Amazon.com's E-Commerce Service (version 4.0). I thought I'd spend a brief amount of time going through the basic motions of getting data from Amazon.com's site. If you want to follow along, I'm assuming you've already signed up for an Amazon.com subscription ID, and you've added a *Web Reference* to their WSDL file in your .NET project.

The first thing to do is add the correct using statement to use Amazon.com's class definitions:

```c#
using AmazonWebServices.com.amazon.webservices;
```

Note that the `AmazonWebServices` part of the namespace comes from the *Default Namespace* property of my project. If your project has a default namespace of "CoffeeGrinders.Products", you would do this:

```c#
using CoffeeGrinders.Products.com.amazon.webservices;
```

For some reason it took me a second to realize what namespace the classes were in that were generated from the WSDL, but once I determined that things started moving quickly from there.

Let's move on to actually getting some data back from Amazon.com. The first thing is to get a reference to a `AWSECommerceService` object:

```c#
AWSECommerceService service = new AWSECommerceService();
```

This object is the gateway to every operation that can be done through the web service API. Virtually every operation I've seen so far follows this pattern:

* Create a request object.
* Populate the request object with the correct information.
* Create a search object that contains your subscription ID and a reference to the request object you just made.
* Pass the search object into the correct service method.
* Check the response object for the results of the operation.

Granted, every service operation is a bit different in terms of functionality and purpose, but they all take the same approach (which is a good thing in my mind - once you get how their API works it's pretty easy to use any other service operation). Let's see how this pattern is applied to finding a customer via their name [1]. First, you create a `CustomerContentSearchRequest` object:

```c#
CustomerContentSearchRequest customerContentSearchRequest 
  = new CustomerContentSearchRequest();
customerContentSearchRequest.Name = this.customerNameTextBox.Text;
customerContentSearchRequest.ResponseGroup = 
  new string[] {"Request", "CustomerInfo"};
```

While every request object will have different request properties (e.g. `Name` and `Email` on `CustomerContentSearchRequest`), all of them have a `ResponseGroup` property that takes an array of `string` objects. Check the API document for information on what values are valid for a given request object as the list varies from request object to request object.

The next part is easy: you create a `CustomerContentSearch` object:

```c#
CustomerContentSearch customerContentSearch = new CustomerContentSearch();
customerContentSearch.SubscriptionId = this.amazonSubscriptionID;
customerContentSearch.Shared = customerContentSearchRequest;
```

The `SubscriptionId` property is the ID given to you by Amazon.com when you sign up to use their web services, and the `Shared` property is set to the request object you just made.

Now you can actually make the request:

```c#
CustomerContentSearchResponse customerContentSearchResponse = 
  service.CustomerContentSearch(customerContentSearch);
```

Each response object has at least two properties: `OperationRequest` (which is of type (you guessed it!) `OperationRequest`) and an array of objects that contains information about the business objects you got back (if any). The only response object I've seen that doesn't follow this pattern exactly is `MultiOperation`, which is used to do a number of service operations in one shot, so the response object has a number of array-based properties, each one corresponding to the business object results. `OperationRequest` gives you information about the operation - i.e. did it succeed? what were the arguments? etc. This property is good for debugging/logging purposes in case you're not getting the response you thought you should for the given request.

The second property takes a bit to get used to. In the customer content search case, you get back an array of `Customers` (note the plural nature of that class name!) objects. Each `Customers` object has an array of `Customer` object (these contain the customer data you're looking for), a `Request` object that has operation status information (the `IsValid` property will tell you if the request was valid), and `TotalPages` and `TotalResults` properties, which are useful when you getting lists of information that can span multiple pages (think Amazon.com's Wish List, where you don't see the entire list at one time; you get a page of 10 items).

That's essentially it. At this point you can traverse your business object arrays and do whatever you need to do with the information. As I've stated before, each service operation has its differences in terms of the request data and the corresponding results. For example, take a look at the following code that gets a random wish list item:

```c#
//  The following method is not shown -
//  it's basically returning a cached value
//  of the number of pages that are in the wish list.
int totalPageCount = this.GetWishListPageCount();

Random r = new Random();
int itemIndex = r.Next(0, 10);
int pageIndex = r.Next(1, totalPageCount + 1);

AWSECommerceService service = new AWSECommerceService();		
ListLookupRequest listLookupRequest = new ListLookupRequest();
listLookupRequest.ListId = "CustomersWishListIDGoesHere";
listLookupRequest.ListType = ListLookupRequestListType.WishList;
listLookupRequest.ListTypeSpecified = true;
listLookupRequest.ProductPage = pageIndex;
listLookupRequest.ResponseGroup =
  new string[] {"Request", "ListInfo", "ItemAttributes", "Large", 
  "ListItems", "ListFull"};

ListLookup listLookup = new ListLookup();
listLookup.SubscriptionId = "SubscriptionIDGoesHere";
listLookup.Shared = listLookupRequest;

ListLookupResponse listLookupResponse = 
  service.ListLookup(listLookup);

ListItem[] items = listLookupResponse.Lists[0].List[0].ListItem;

if(itemIndex > (items.Length - 1))
{
  itemIndex = items.Length - 1;
}

Item item = items[itemIndex].Item;

this.itemPageLabel.Text = pageIndex.ToString();
this.itemIndexLabel.Text = itemIndex.ToString();
this.itemAsinLabel.Text = item.ASIN;
this.itemTitleLabel.Text = item.ItemAttributes.Title;
```

The pattern is the same as a customer lookup, but the are definitely data differences in both the reqest and respons objects (especially in the resulting information as it can be a bit overwhelming to digest). The `ItemAttributes` property on an `Item` has over 200 properties to examine! The API designers basically lumped everything that they could think of that an item could have and put it all in one type definition. Unfortunately, that means that there's a `TotalGemWeight` property on an `Item` object that represents a book ... which really doesn't make a whole lot of sense, but generally you'd just ignore those property values that you know don't apply for the item you have [2].

I hope this has been helpful if you're just starting out (like I am!) with Amazon.com's web services. I took a look at their API around version 2.0 and I'm happier with the design in 4.0 as it feels more consistent. It takes a bit to get used to, but once you understand the approach they use it's clear sailing from there. If I run into any other issues or ideas with their API in the future I'll post my observations.

[1] I should point out that I'd like to search via an e-mail address, but for some reason I can't get it to work with my own e-mail address. I can find myself via my name, but the e-mail address is unique, so I'd always find that specific customer (or none if the e-mail address isn't in Amazon.com's database). However, I never get my customer record back when I go that route. If you've seen the same problem and have some insight into this issue please post a comment - thanks in advance.

[2] Note that the `ProductGroup` and `Brand` properties are useful in determining what kind of item you have.

> Published: 01.28.2005 12:34:10 PM CST