---
title: Preserving Order With Java's LinkedHashMap Class
path: /preserving-order-with-javas-linkedhashmap-class
date: 2015-06-07
summary: Preserving Order With Java's LinkedHashMap Class
tags: ["CFML", "Java"]
---

The other night I ran into an issue with preserving order of keys and values being added to some ColdFusion structures. As it would be, structures in CFML don't preserve data in any consistent order. It's honestly something I've known about, but none the less, something that's never given me hassle. With that faintly in the back of my mind, I did not accept it as such initially as when I wrote my code and ran it in ColdFusion 11, everything appeared in order (probably not the best route of thinking). It wasn't until I started testing in Lucee that I realized it was all a chance of luck in how my data was being displayed to screen. In fact, further examination proved that the order was not as correct as I thought originally with ColdFusion 11 either.

#### So here's my scenario...

I've been building out various features for an archive section of my blog. As a index to that page, I wanted to capture all of my posts and display them in separated collections; ordered by [ year / month / article ]. In my case, I use ORM. So when grabbing blog posts, I'm getting an array of article objects. I loop over the array and populate a structure with the year as the key and the value being another structure. That structure is then populated with the month as it's key and an array of articles as it's value. I'll articulate this better below. I'll be using a struct of dummy articles to represent my article objects.

#### Consider this code...

```java
<cfscript>
	// Some dummy articles yay!
	articles = [
		{title: "I Am a Title Perhaps", publishDate: createDate(2015, 11, 08)},
		{title: "I Am a Title As Well", publishDate: createDate(2014, 08, 15)},
		{title: "I Am a Title I Think", publishDate: createDate(2014, 09, 25)},
		{title: "I Am a Title", publishDate: createDate(2013, 04, 05)},
		{title: "I Am a Title Too", publishDate: createDate(2013, 05, 17)},
		{title: "I Am a Title Also", publishDate: createDate(2013, 07, 02)}
	];
	
	collection = {};
	for (post in articles) {
		year = year(post.publishDate);
		if (!structKeyExists(collection, year)) {
			collection[year] = {};
		}
		if (!structKeyExists(collection[year], monthAsString(month(post.publishDate)))) {
			collection[year][monthAsString(month(post.publishDate))] = [];
		}
		arrayAppend(
			collection[year][monthAsString(month(post.publishDate))],
			{"title": post.title, "date": post.publishDate}
		);
	}
	
	writeDump(collection);
</cfscript>

<cfoutput>
<cfloop item="year" collection="#collection#">
	<h4>#year#</h4>
	<cfloop item="month" collection="#collection[year]#">
		<div><strong>#month#</strong></div>
		<cfloop index="article" array="#collection[year][month]#">
		    <div>#article.title# | #dateFormat(article.date, "YYYY/MM/DD")#</div>
		</cfloop>
	</cfloop>
</cfloop>
</cfoutput>
```

*Try it out:*

[ColdFusion 11 - Example](http://trycf.com/editor/gist/6d678f1a7800423a9484/acf11)

[Lucee 4.5 - Example](http://trycf.com/editor/gist/6d678f1a7800423a9484/lucee)

With ColdFusion 11, the years appear to hold a sensible order; however, the months are not so well off. Lucee on the other hand twists things all around. This was definitely not what I was trying to achieve. Luckily there is a simple, solid solution.

I had taken to Twitter the next morning where I got some great insight and suggestions from James Moberg, Adam Cameron and John Whish (thanks fellas!). John brought up a nifty feature in Lucee (and Railo) where you can preserve order in a struct by using structNew("linked"). This is cool but I've been working on a set of code that could see some cross platform usage so I needed something mutually supported. John then suggested I try [Java's LinkedHashMap](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) class and gave me reference to an [article by Jeremy Gibbens](http://www.aftergeek.com/2010/03/preserving-structure-sort-order-in.html).

This did the trick! Better yet, you can (as far as I can tell) treat a LinkedHashMap exactly like a CFML structure; functions and all. With that said, I didn't need to modify any of the current code really; except for where I needed to apply the LinkedHashMap instead of a normal struct.

#### Check out the solution...

```java
<cfscript>
	// Some dummy articles yay!
	articles = [
		{title: "I Am a Title Perhaps", publishDate: createDate(2015, 11, 08)},
		{title: "I Am a Title As Well", publishDate: createDate(2014, 08, 15)},
		{title: "I Am a Title I Think", publishDate: createDate(2014, 09, 25)},
		{title: "I Am a Title", publishDate: createDate(2013, 04, 05)},
		{title: "I Am a Title Too", publishDate: createDate(2013, 05, 17)},
		{title: "I Am a Title Also", publishDate: createDate(2013, 07, 02)}
	];
	
	LinkedHashMap = createObject("java", "java.util.LinkedHashMap");
	collection = LinkedHashMap.init();
	for (post in articles) {
		year = year(post.publishDate);
		if (!structKeyExists(collection, year)) {
			collection[year] = LinkedHashMap.init();
		}
		if (!structKeyExists(collection[year], monthAsString(month(post.publishDate)))) {
			collection[year][monthAsString(month(post.publishDate))] = [];
		}
		arrayAppend(
			collection[year][monthAsString(month(post.publishDate))],
			{"title": post.title, "date": post.publishDate}
		);
	}
	
	writeDump(collection);
</cfscript>

<cfoutput>
<cfloop item="year" collection="#collection#">
	<h4>#year#</h4>
	<cfloop item="month" collection="#collection[year]#">
		<div><strong>#month#</strong></div>
		<cfloop index="article" array="#collection[year][month]#">
		    <div>#article.title# | #dateFormat(article.date, "YYYY/MM/DD")#</div>
		</cfloop>
	</cfloop>
</cfloop>
</cfoutput>
```

*Try it out:*

[ColdFusion 11 - Example 2](http://trycf.com/editor/gist/3c2e508e8b5fcd05a66c/acf11)

[Lucee 4.5 - Example 2](http://trycf.com/editor/gist/3c2e508e8b5fcd05a66c/lucee)

From the output above, now I'm getting the order I was looking for. Pretty cool!

Do note that when displaying the data from a LinkedHashMap with cfdump, you might not see the desired order; but when actually printing out the data, it will retain the order in which it was first constructed. Also keep in mind that a LinkedHashMap is case sensitive!

Cheers.