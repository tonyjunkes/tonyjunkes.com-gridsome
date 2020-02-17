---
title: Railo / Tomcat 8 Error & Solution For (A child container failed during start)
path: /railo-tomcat-8-error-and-solution-for-a-child-container-failed-during-start
date: 2014-11-14
summary: Railo / Tomcat 8 Error & Solution For (A child container failed during start)
tags: ["CFML", "Apache Tomcat"]
---

I noticed a few days ago that a new update for Apache Tomcat 8 was released (8.0.15). Today I decided to go ahead and update my server but ran into a minor hiccup when trying to access my websites tied to Railo.

Usually, updating Tomcat 8 with one of it's minors is pretty straight forward. Download the new version, unzip and swap out the JAR files in the "lib" folder and restart Railo/Tomcat. Going from 8.0.14 to 8.0.15, not so much. At least for me. Everything in the console came back ok when I restarted Railo however, after checking one of my sites, I noticed I was getting 502's with no recovery.

To the Tomcat logs I went. Specifically the catalina.out log. What I found was a chain of exceptions that ultimately told me 2 things - Tomcat was failing to start and, after brief digging in those traces, it was due to a missing class.

Here's what I found:

`SEVERE: A child container failed during start java.util.concurrent.ExecutionException: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Catalina].StandardHost[my domain here].StandardContext[]]`

A little further down the stack revealed this...

`Caused by: java.lang.ClassNotFoundException: org.apache.juli.WebappProperties`

Which (after some googling) is a class inside of the tomcat-juli.jar in Tomcat's "bin" folder. This is not something I normally update; so I updated the JAR file, restarted Railo/Tomcat and, like magic, all of my sites were functioning as usual.

So there we have it. Hopefully this helps someone else out. An issue like this is definitely not specific to Railo so it could possibly strike any application running through Tomcat 8 after an update.

Happy coding!