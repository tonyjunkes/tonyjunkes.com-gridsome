---
title: SES URLs In FW/1 With Routes & Mod_Rewrite
path: /ses-urls-in-fw1-with-routes-and-mod-rewrite
date: 2013-06-22
summary: SES URLs In FW/1 With Routes & Mod_Rewrite
tags: ["ColdFusion", "FW1", "Apache Mod_Rewrite"]
---

After switching my site's overall architecture from Mango Blog to Framework One, some readers have shown interest in what I've done to build certain aspects and areas of the site's structure. First of all, that's pretty cool to me that people are interested at all so I'm down for explaining what I can about what I have done so far (there's a lot still pieced together). In coming posts with this one I'll be breaking down certain areas of my FW/1 structure and the logic I run with. One of the questions was how I achieved the SES URLs that I use so I'm going to start with that today.

In a nutshell, I use a mix of Apache's Mod_Rewrite and FW/1's URL Routes. In the case of say, a blog post or product listing, I would store the title as a nice URL safe string in the database for data lookups via a parameter value in the URL scope. An example of such a string value would be "this-is-my-blog-post". That's the concept so let's look over the grit of it.

### Examples

The base that I typically run with all of the FW/1 apps I've built will include a .htaccess file that contains at least a rewrite for URLs to not include index.cfm. The URLs look better to me personally; though I have found some people favor to keep that part.

This is what that file would contain so far which will rewrite `mysite.com/index.cfm/page` to `mysite.com/page`.

```md
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} !^.*\.(bmp|css|gif|htc|html?|ico|jpe?g|js|pdf|png|swf|txt|xml)$
RewriteRule ^(.*)$ index.cfm/$1 [NS]
```

The next rewrite is something I've written about briefly before which is [rewriting a FW/1 Subsystem URL to be a SES URL](http://tonyjunkes.com/blog/cleaner-fw-1-subsystem-url-with-apache-mod-rewrite).

Here's the updated .htaccess which will rewrite `mysite.com/index.cfm/subsystem:section` to `mysite.com/subsystem/section`.

```md
RewriteEngine On
# Rewrite FW/1 Subsystem
RewriteRule ^admin/(.*)$ index.cfm/admin:$1 [L]
# Rewrite index.cfm
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} !^.*\.(bmp|css|gif|htc|html?|ico|jpe?g|js|pdf|png|swf|txt|xml)$
RewriteRule ^(.*)$ index.cfm/$1 [NS]
```

Let's say I have a FW/1 page for displaying a blog post. This part of the app would have a URL that consists of - `domain/section/item/parameter/value` or, for example, `mysite.com/blog/show/post/my-blog-post`. From the get go that's not a bad URL but let's say I want to make it look like this - "mysite.com/blog/my-blog-post". For a while I was using Mod_Rewrite to achieve this.

Which would look like this. . .

```md
RewriteEngine On
RewriteRule ^blog/.*$ blog/show/post/$1
```

Then I had a RTFM moment scanning through the [Framework One Developing Applications Manual](https://github.com/framework-one/fw1/wiki/Developing-Applications-Manual) and came across URL routes. These can be set in the VARIABLES.framework.routes setting within the Application.cfc. From a glance, you can take the original URL and route it to a prettier looking URL for the same visual achievement as using Mod_Rewrite.

Consider this code. . .

```js
component extends="org.corfield.framework"
  output="false"
{
	/*
	...Some code...
	*/

	VARIABLES.framework = {
		routes = [
			{ "/section/:value" = "/section/item/parameter/:value" } //Example - "/blog/:post" = "/blog/show/post/:post"
		]
    	//Other framework vars would also be set here.
	};

	/*
	...Some more code...
	*/
}
```

### Final Touches

When I have a URL like "mysite.com/blog/my-blog-post", the last portion of the of the URL string is the parameter value which is checked against the database to pull the referencing field in the table containing the blog post. What I do is actually store that string in a column called `post_url` as the reference. Then all I have to do is pull back the record where post_url equals the parameter value.

I've never been all that good at the whole URL rewriting bit (considering lack of regex knowledge); but the more I've pushed through writing these types of apps, the better I have become at it. FW/1 just happens to make it even easier to approach SES URLS from the start. No regex needed. Couple a few related approaches though and you have things marrying together even better.

I'm sure there are some other alternatives to tackling certain parts; but I hope these examples have given some solid insight and a good medium to start on.

Until the next installment.

Cheers.