---
title: A Brief Look at the Rewrite Valve in Tomcat 8
path: /a-brief-look-at-the-rewrite-valve-in-tomcat-8
date: 2015-09-11
summary: A Brief Look at the Rewrite Valve in Tomcat 8
tags: ["Lucee", "Apache Tomcat", "URL Rewrites"]
---

Recently I learned that Tomcat 8 has its own built in approach to handling URL rewriting to achieve things like SES (pretty) URLs. Personally I do URL rewrites from Nginx, as it's the forward facing web server and proxy to all of my Lucee apps that live in Tomcat; but, for the sake of learning, I wanted to take a look at what it had to offer.

## Setting it up...

Reading through the [Tomcat 8 Rewrite Docs](http://tomcat.apache.org/tomcat-8.0-doc/rewrite.html), I was delighted to see that they use a similar approach to syntax as Apache's Mod_Rewrite so the learning curve of writing actual rewrites is minimal to anyone who's worked with Apache in the past.

Essentially, all you need to do is include the rewrite valve class `org.apache.catalina.valves.rewrite.RewriteValve` in your application's context. This can be either the global context.xml or in the context block of a host in the `server.xml`; both found in Tomcat's `/conf` directory. Then drop a `rewrite.config` file containing your rewrites into the WEB-INF folder of ROOT or wherever your application's root `WEB-INF` is. Using the global `context.xml` will effect all virtual host setups you've defined in your `server.xml` so if you have multiple apps running, it may be best to do a per host setup of the rewrite valve.

#### The global approach:

Here's an example of setting the valve in Tomcat's `context.xml`

```xml
<?xml version='1.0' encoding='utf-8'?>
<!-- The contents of this file will be loaded for each web application -->
<Context>
    <!-- REWRITE VALVE -->
    <Valve className="org.apache.catalina.valves.rewrite.RewriteValve" />
    <!-- // -->
  
    <!-- Speed up context loading -->
    <JarScanner scanClassPath="false" />
    <!-- Default set of monitored resources. If one of these changes, the -->
    <!-- web application will be reloaded.                                -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
</Context>
```

#### The individual host approach:

In my case with running multiple Lucee applications through Tomcat, I would want to set up the rewrites per host. Here's an example of a host block in the `server.xml`.

```xml
<Host name="tonyjunkes.com" appBase="webapps" unpackWARs="false" autoDeploy="false">
  <Context path="" docBase="C:/websites/tonyjunkes/www">
    <Valve className="org.apache.catalina.valves.rewrite.RewriteValve" />
  </Context>
</Host>
```

#### Making it all happen with rewrite.config

If you're running your application from the base ROOT of Tomcat or are using a global rewrite to effect all virtual hosts, then you can drop your rewrite.config right into the WEB-INF there. Otherwise you will need to locate the WEB-INF of your application. In the case of my Lucee apps, the WEB-INF will be in my application's web root.

Here's an example rewrite.config that rewrites the URL to exclude index.cfm.

```md
RewriteCond %{REQUEST_URI} !^.*\.(bmp|css|gif|htc|html?|ico|jpe?g|js|pdf|png|swf|txt|xml|svg|eot|woff|woff2|ttf)$
RewriteRule ^(.*)$ index.cfm/$1 [L]
```

There we have it. Nice and simple. I haven't done much else with it at this point so I'm not aware of any major caveats with intricate rewrites.

[Webucator](https://www.webucator.com/), a provider of [Tomcat Online and Onsite Training Classes](https://www.webucator.com/servers/tomcat.cfm), has produced a video demonstrating the use of Tomcat's Rewrite Valve as I've explained above.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZjyhvLyHZfI" frameborder="0" allowfullscreen></iframe>

Cheers and happy coding!