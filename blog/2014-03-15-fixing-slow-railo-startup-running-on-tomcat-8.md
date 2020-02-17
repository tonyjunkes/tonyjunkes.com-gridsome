---
title: Fixing Slow Railo Startup Running on Tomcat 8
path: /fixing-slow-railo-startup-running-on-tomcat-8
date: 2014-03-15
summary: Fixing Slow Railo Startup Running on Tomcat 8
tags: ["CFML", "Lucee", "Apache Tomcat"]
---

Over the last week++ I have been taking on the role of Server Admin. I by no means consider myself to be one, let alone good at it; but, since I manage my own VPS through Viviotech, it has deeply become a necessity where there comes that time when I need to be that guy. Quite often more than less.

## My situation...

As of late I've been battling frequent occurrences where my server gets brought to it's knees from hits by various bots; among other things. In some cases the hang would wig out a request and crash the expected result ending in a barrage of errors. Page requests would take 5-10mins for every single request! After playing with how I was distributing memory and applying some better handling of objects and sessions, things have been running smooth. From there I decided I wanted to go the extra step and show some love to the stack of the server side. So I upgraded Java to the latest (7u51) and Tomcat 7.0.52 to version 8.0(.3).

Upgrading Tomcat in itself was a bit of a tug but nothing insane. I was hoping it would be a "just copy over the /lib folder" install but it wasn't ;) I suspect the updates to some of the config files was the culprit but I don't know for sure as I simply merged everything and re-added the bits in the XML that make Railo and Tomcat talk. If/When I get time I will shed more light on this; but that's another blog post!

Upon getting things settled and sites rendering again I instantly hit a noticeable drawback - really slow server startup. In my case about 1-2 minutes! All in all, I host 7 sites of a small to moderate size with no steep requirements at start/render time. This delay was a smack in the face difference. So I took a look in the catalina.out log file and came across a few warnings.

## The solution among some other fixes...

Take note that some of these warnings probably/definitely do not effect performance, if at all. I'm simply going to knock 'em out here as I did at the time.

#### Update: If you use Lucee, these fixes also apply! :)

Let's start with the main fix of lowering the start time of our sites. With every site context that was loaded, I would get this response in the logs. . .

`Mar 09, 2014 11:55:07 PM org.apache.jasper.servlet.TldScanner scanJars INFO: At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.`

A Google search brought me to [wiki.apache.org](http://wiki.apache.org/tomcat/HowTo/FasterStartUp) which states that TLD Scanning checks JAR files for tag libraries and their descriptor files. Everywhere I read more in regards to this scan pointed out that startup time can take a nasty blow; as this scan is performed on each context. Seeing as I only run CFML apps with Railo, it seemed ok to completely turn scanning off.

I found that there are a few documented ways to turn this scan off; however, I could only get two ways to work for me. Many results on Google, including a StackOverflow question, said to set the `processTlds` property to `false` in the `<Context>` of a given site setup in server.xml. When I tried this I got no results and a warning in catalina.out stating there was no such property. To confuse me more, the property is listed in the Tomcat 8 docs giving me the impression that you can use it. I'm not completely sure in that regard though. Oh well.

#### First solution:

A simple, surefire method of switching the check off is to go into `context.xml`, located in Tomcat's /conf directory, and add `<JarScanner scanClassPath="false" />`. That's it!

#### Second solution:

Inside catalina.properties, located in the /conf directory, around line: 97, is a setting declared for `tomcat.util.scan.StandardJarScanFilter.jarsToSkip`. In it's original form there is a list of select JARs to be skipped. What I tried doing was commenting out that list and replacing it with `*.jar` giving this line. . .

`tomcat.util.scan.StandardJarScanFilter.jarsToSkip=*.jar`

Nice!

With either changes alone, after a fresh restart, I was getting a startup time averaging around 8 seconds and no more warning of JAR scanning in catalina.out. That's 60+ seconds down to ~8. Awesome!

So let's go over the other warnings. Next was this...

`SLF4J: Class path contains multiple SLF4J bindings. SLF4J: Found binding in [jar:file:/opt/railo/lib/slf4j-nop.jar!/org/slf4j/impl/StaticLoggerBinder.class] SLF4J: Found binding in [jar:file:/opt/railo/lib/railo-sl4j.jar!/org/slf4j/impl/StaticLoggerBinder.class] SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.`

From what I have gathered from searching, the Railo folks switched `railo-sl4j.jar` out for `sl4j-nop.jar` after version 4. This is a result of me upgrading Railo 3.x to 4 once upon a time and never noticing I guess. Removing `railo-sl4j.jar` kills the warning and things function all the same (It's just one less error in the logs really).

The last occurrence was this. . .

`Mar 09, 2014 11:54:36 PM org.apache.catalina.users.MemoryUserDatabase open WARNING: Exception configuring digester to permit java encoding names in XML files. Only IANA encoding names will be supported.`

Again, trusty Google landed me at [http://adfinmunich.blogspot.com/2013/04/exception-during-startup-from-tomcat-6.html](http://adfinmunich.blogspot.com/2013/04/exception-during-startup-from-tomcat-6.html) which gives an explanation and a list of properties to add to `catalina.properties`.

`javax.xml.parsers.DocumentBuilderFactory=com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl
javax.xml.transform.TransformerFactory=com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl
javax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl
javax.xml.datatype.DatatypeFactory=com.sun.org.apache.xerces.internal.jaxp.datatype.DatatypeFactoryImpl`

After restart there was no more occurrence of the warning and things continued to function.

#### Update: 03/17/14 - Some things I forgot. . .

Depending on what version of Tomcat you are using (It could vary from what I've noticed), there are some directories that you can do away with in the "/webapps" folder inside of Tomcat. These dirs include /docs, /examples, /host-manager and /manager (in Tomcat 8 at least). Make sure to leave "/ROOT". The docs folder is filler as I see it and the other 3 are processed during Tomcat's startup. This will incur a check through each WEB-INF dir; especially if Jar Scanning is on. I've had them removed for some time now and seen no issue since.

## Some things I haven't looked into. . .

The above helped me a great deal. Hopefully it helps someone else. There are some other bits that I've started to look into regarding performance at startup but have yet to dive into and test.

The wiki.apache.org link is a good start for things to try/modify/apply. I have also read mixed opinions on applying the "autoDeploy" and "unpackWars" properties to "false" in a site's `<Host>` element in `server.xml`. From what I read if there is any performance gain, especially considering you might not even have a WAR file to unpack or deploy, it's minimal with today's rigs.

I have also come across loading website contexts in parallel threads at startup by setting "startStopThreads" to 1 or greater in `<Host>`. This apparently can give a boost when applied correctly. No experience on this though.

The Tomcat 8 docs on these properties can be found here.

As I learn more on tuning and the lot I will be sure to throw another post together. Cheers!