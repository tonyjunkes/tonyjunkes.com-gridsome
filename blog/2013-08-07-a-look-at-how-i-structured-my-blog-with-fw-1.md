---
title: A Look At How I Structured My Blog With FW/1
path: /a-look-at-how-i-structured-my-blog-with-fw-1
date: 2013-08-07
summary: A Look At How I Structured My Blog With FW/1
tags: ["ColdFusion", "CFML", "FW1"]
---

#### Note: This post is outdated based on how FW/1 and my structuring practices have evolved. While there may be some take-away from the article, I don't consider this a solid FW/1 structure anymore.

It's been a hot minute since I last posted anything. Today I wanted to touch more on how I use Framework One with my blog; since my last post - [SES URLs In FW/1 with Routes and Apache Mod_Rewrite](http://tonyjunkes.com/blog/ses-urls-in-fw1-with-routes-and-mod-rewrite). This time I'm going to describe the basic layout of my application structure.

A word to the wise, I am not wise. I'm still grasping a mix of methodologies in building a solid application along with Framework One. I am always open to suggestions or being asked - WTF are you doing that for?

If you are not familiar with Framework One, I highly recommend checking out the FW/1 [Developing Applications Manual](https://github.com/framework-one/fw1/wiki/Developing-Applications-Manual) and [Reference Manual](https://github.com/framework-one/fw1/wiki/Reference-Manual).

So let's rip apart and take a look at some example directory structures I commonly use. Some of these files and directories are pretty straight forward but I'll go over every one in some detail.

## The Root Level

```md
| ROOT
|--- /assets
|--- /org
|--- /app
|--- Application.cfc
|--- index.cfm
|--- .htaccess
```

#### /org

This is where framework.cfc, the entire code base for FW/1, is kept.

#### Application.cfc

The Application.cfc at the root is the base file to my entire application; that extends FW/1 and sets a variety of application scope variables, FW/1 specific vars, request, session and error handling.

Here's a very plain look at some of what I use in the Application.cfc

```java
component extends="org.corfield.framework"
  output="false"
{
  /****
  NOTE This is very plain structure with not a lot of drop in ready code as it's here to show structure.
  ****/

  //This mapping is for including pod like apps into a page.
  THIS.mappings["/modules"] = ExpandPath("./path/to/modules");
  //The classes dir is for any extra Java bits for the app. For example - I use BCrypt sometimes.
  THIS.javaSettings = {LoadPaths = ["./path/to/classes"], watchExtensions = "class"};

  //For more on FW/1s framework vars see https://github.com/framework-one/fw1/wiki/Reference-Manual
  VARIABLES.framework = {
	base = "/app",
  	action = 'page',
	usingSubsystems = true,
	defaultSubsystem = 'public',
	siteWideLayoutSubsystem = 'common',
  	generateSES = true,
  	SESOmitIndex = true,
  	reload = 'refresh',
  	password = 'password',
  	routes = [
   		//FW/1 Routes
		{ "/section/parameter/:parameter" = "/section/item/parameter/:parameter" }
	]
  };

  public function setupApplication() {
  	//Application Scope Vars
  }

  public function setupRequest() {
	//OnRequest handling.
  }

  public function setupSession() {
	//OnSessionStart handling.
  }

  public function onMissingView(struct rc = {}) {
	//Handle views/pages that do not exist. 404 handling etc.
  }

  public void function onError(any exception, string event) {
	//error handling.
  }
}
```

#### index.cfm

This is a blank file used by the framework. All requests and output is routed through this file.

#### /assets

This is where I break down dirs for CSS, JavaScript and Site-wide Image files.

#### .htaccess

Any URL rewrites that I don't handle in the Application.cfc are handled here using Apache Mod_Rewrite.

#### /app

This is the heart of my application. I like to set the base location in FW/1 to another directory (like so) to separate the innards of the application from the main level where site-wide content may be held (assets, Sitemap.xml etc.). My public files, admin, helper files and CFCs are all kept in this directory.

## The Framework One Base Location - /app

```md
| /app
|--- .htaccess
|--- /admin
|--- /public
|--- /classes
|--- /models
|--- /modules
|--- /common
|--- /utils
```

#### .htaccess

I use this file simply to apply a deny-all to restrict access to the /app directory from a browser.

#### /public & /admin

These are both Framework One Subsystems. Subsystems work as mini apps within the entire application. I define /public to be the default base location for FW/1 to look for controllers and views for displaying public content via the root Application.cfc. Both directories contain /controllers and /views folders, where sub-folders are kept within those, pertaining to given pages and actions. In the case of /admin, this area and all of the website pages referenced to it are locked behind the requirement of user authentication.

#### /common

When Subsystems are active in FW/1, the /common directory acts as a shared base for layouts used through the application. The /layouts sub-folder houses a default.cfm that contains the HTML for my site-wide design of the blog.

#### /classes

This is where I keep any Java classes I might use with the application. For example, I use BCrypt a lot; though I've added that to the server class path so it's globally accessible by all of my website applications. All class files are dynamically loaded in via THIS.javaSettings in the Application.cfc.

#### /models

The /models directory is a series of sub-folders housing the logic of the application in different forms. First, a "domains" dir for CFCs that hold the structure logic of any objects created from database tables. These objects consist of properties that relate to the table columns and represent the shell to be filled by a service etc. Next, a "services" dir that holds CFCs with a assortment of functions used by the controllers to accomplish a request; be it conversions, data requests etc. This leads to the "gateways" dir which contains CFCs that act as makeshift DAOs for CRUD and data retrieval. The gateways are the only objects that touch the database.

The /models directory is a work in progress as I am doing a messy play of OO and basing inspiration from other frameworks like Grails. I've also started to throw in some dependency injection as I've built up so many services that are tying to the database and others that are merely helper functions. Some structured management would help.

#### /modules

This is where I keep widget like apps or pods. For example I have modules for retrieving content and creating the display structure to the "recent articles" and "recent comments" in the sidebar. This separates them nicely so that I can include them more or less anywhere. This directory is mapped for the application from THIS.mappings in the Application.cfc.

#### /utils

As typically used, I keep utility functions and CFCs here. For example I have components for form validation, formatting etc.

## There we have it ;)

That's a rough idea on how I've laid out the overall structure of the blog. So much of it has been rigged together though that I change and re-factor constantly. For the better. I didn't touch on much code but this structure is what helps the application make sense and flow so far. Hopefully this gives some ideas or direction on how to take your FW/1 app. As I progress, I definitely want to follow up with some code examples. I just haven't crossed anything that seemed worthy.

Cheers.