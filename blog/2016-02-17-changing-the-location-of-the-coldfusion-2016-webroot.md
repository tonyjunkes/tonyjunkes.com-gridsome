---
title: Changing the Location of the ColdFusion 2016 Webroot
path: /changing-the-location-of-the-coldfusion-2016-webroot
date: 2016-02-17
summary: Changing the Location of the ColdFusion 2016 Webroot
tags: ["ColdFusion", "Apache Tomcat"]
---

Alrighty. First blog post of the year (and for a while). Better late than never right?

So Adobe have slipped their newest version of ColdFusion, ColdFusion 2016, into the wild. My opinion of it is kind of _meh_. After participating in the pre-release, I've realized there's really nothing in it for me that would make me switch. I have one client on ColdFusion 11 and if I was to move them any which way in the CFML world, it'd probably be in the direction of Lucee; which is what all of my other projects use. That's a whole other topic though ;)

There is one bit of info I wanted to pass on that ties into ColdFusion 2016 and the version of Apache Tomcat it uses, Tomcat 8. Changing the location of the Webroot. When I first found out that CF2016 was using Tomcat 8 I was quite delighted. There's a fair amount of newer features and changes compared to Tomcat 7; though I don't know how much of that is lost since Adobe have hacked apart the vanilla install of Tomcat to work with ColdFusion.

Since version 10, ColdFusion has used various releases of Tomcat 7. The method for changing the location of the Webroot then was to use the "aliases" attribute within a Context element of server.xml as explained by both [Ryan Anklam](http://blog.bittersweetryan.com/2012/02/changing-webroot-of-coldfusion-zeus.html) and [Nando Breiter](http://dnando.github.io/blog/2014/12/04/change-location-of-cf11-webroot/) in their blog posts. With the coming of version 8 however, this method has been deprecated and removed. The "new" way is to pass the location in a Resources element.

Here's an example of changing the location of the Webroot for ColdFusion 2016 in Tomcat 8 from within the Host element of server.xml...

```xml
<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="false">
    <Context path="" docBase="C:\websites\coldfusion2016\wwwroot" workDir="C:\ColdFusion2016\cfusion\runtime\conf\Catalina\localhost\tmp">
          <Resources>
              <PreResources className="org.apache.catalina.webresources.DirResourceSet" base="C:\ColdFusion2016\cfusion\wwwroot\CFIDE" webAppMount="/CFIDE" />
              <PreResources className="org.apache.catalina.webresources.DirResourceSet" base="C:\ColdFusion2016\cfusion\wwwroot\WEB-INF" webAppMount="/WEB-INF" />
          </Resources>
    </Context>
</Host>
```

That's it for now. Cheers.