---
title: Exposing Private Fields in Jsoup's Whitelist Class with Reflection
path: /exposing-private-fields-in-jsoups-whitelist-class-with-reflection
date: 2015-06-18
summary: Exposing Private Fields in Jsoup's Whitelist Class with Reflection
tags: ["CFML", "jsoup", "Java", "Stack Overflow"]
---

In recent days I've been trying to knock out some answers to questions on Stack Overflow. I've actually been pretty successful in helping some people out, so I'm happy about that.

Two of the questions I ended up answering were in regards to using ColdFusion with [Jsoup, a Java based Document Parser](http://jsoup.org/), and it's Whitelist class in some strange different ways. By that I mean they wanted to gain access to data that isn't freely exposed; and in most cases, I feel you really don't need it to be.

Here's the questions:

[JSOUP - How to get list of disallowed tags found in html?](http://stackoverflow.com/questions/30817745/jsoup-how-to-get-list-of-disallowed-tags-found-in-html)

[How to get list of valid tags of Jsoup whitelist?](http://stackoverflow.com/questions/30841717/how-to-get-list-of-valid-tags-of-jsoup-whitelist)

The first question was a pretty fair concept (I could see possibly using it) and overall easy to implement. The second question was slightly trickier because in order to do what the OP was asking, you'd have to access the private fields in the Whitelist Java class. In order to do this, an immediate remedy is to use Java's reflection methods. After a little fiddling I had a working example; but I didn't stop there. In the end, I figured since I dug into one field, why not get at the others?

So I created a CFC with a collection of helper functions to get at the innards of Jsoup's Whitelist class. In case anyone else may be interested, [I put it up on Github: Jsoup Whitelist Helper](https://github.com/tonyjunkes/Jsoup-Whitelist-Helper).

Cheers.