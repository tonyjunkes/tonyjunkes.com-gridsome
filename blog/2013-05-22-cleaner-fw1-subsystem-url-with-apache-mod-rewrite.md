---
title: Cleaner FW/1 Subsystem URL With Apache Mod_Rewrite
path: /cleaner-fw1-subsystem-url-with-apache-mod-rewrite
date: 2013-05-22
summary: Cleaner FW/1 Subsystem URL With Apache Mod_Rewrite
tags: ["ColdFusion", "FW1", "Apache Mod_Rewrite"]
---

This is more of a reference for myself but if anyone is as bad as I am at URL rewriting (and regex for that matter) then this could be of some help.

I use Apache for my dev and production so naturally when I'm looking to do SES URLs I'm digging into mod_rewrite. In the most general of uses, I usually go searching for examples that relate to my given scenario; which is pretty straight forward as far as what I'm trying to do. Someone has had to have done it. Typically when I search the almighty Google I use a reference to PHP for the sake of getting as many examples as I can.

In the case of this site for example, when I wanted to do some rewriting, I searched in relation to Mango Blog and found some forum/blog posts; copied them and bended them as I needed and all was well.

With FW/1 I had been trying to rewrite subsystem URLs for a while on my own; and was failing miserably with no solid result. Now of course, I could have reached out to the community or Google Groups in regards to this but I was really trying to grasp this on my own. In the end I used a makeshift of a rewrite I use with the URLs on this site.

As an example, what I could be working with would look like this - `http://mysite.com/index.cfm/admin:main`.

I'm a bit anal about the "index.cfm" and the ":". I wanted to remove both and make the URL look like this - `http://mysite.com/admin/main`.

This is what did the trick:

```md
RewriteEngine On
RewriteRule ^admin/(.*)$ index.cfm/admin:$1 [L]
```

Now this might have seemed simple to some but I am a complete noob to the matter; and the fact that I got something working is a big deal even if it was as plain as this might actually be.

Anyway, happy coding.