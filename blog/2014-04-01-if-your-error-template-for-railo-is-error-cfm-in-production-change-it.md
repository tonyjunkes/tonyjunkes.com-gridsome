---
title: If Your Error Template For Railo Is error.cfm In Production, Change It!
path: /if-your-error-template-for-railo-is-error-cfm-in-production-change-it
date: 2014-04-01
summary: If Your Error Template For Railo Is error.cfm In Production, Change It!
tags: ["CFML","Lucee","Security"]
---

> Note: This applies for Lucee as well.

So not too long ago [Brad Wood had tweeted](https://twitter.com/bdw429s/status/448935313097117696) about changing Railo's default error template from error.cfm to error-public.cfm. I wanted to take a moment and echo this again.

For the sake of security, change it.

This can easily be changed in the Railo Adminstrator by going to `Settings -> Error`. From there change `General Error Template` and `Missing Template Error` from `error.cfm` to `error-public.cfm`.

In a varying situation, this can be a trivial issue; yet also manifest into something that comes back to bite you. How hard can vary.

The issue with error.cfm is that it renders a default display of the error that occurred. This is where problems can arise. Depending on the error and the data involved, you're at least guaranteed right from the get go that this dump will display to the world info like - what version of Railo you're running and a directory structure that shows the base path to the file that errors. Generally, this info could be trivial; however, should an exploit to your version enter the wild, you now display a welcome card stating you are potentially vulnerable.

Now if you run your application with a Application.cfc using onError() you help steer off the chance of such an error displaying; however, not completely. An example of where your Application.cfc may not catch an incoming error is with the use of CAPTCHAs with CFIMAGE.

Because the image generated is only temporarily used, it gets grabbed from the temp directory by Railo's /context/graph.cfm for display to the user and then discarded. Suppose the page being requested though is a cached page. Chances are pretty good that when this cache is requested, the image has already been tossed. It doesn't exist. Now you end up with an error (a trivial one mind you) like this in your logs - `file or directory /path/to/your/website/WEB-INF/railo/temp//graph/6DE59325-0F57-4375-AAE531EDEB7CCE09.png not exist`. Inside of the `/context` folder is a Application.cfm that negates the parent Application.cfc from catching the error. This will result in Railo's base error template catching it; and with error.cfm being set, the user will receive a raw display of the error.

A relative Google search actually yields a handful of sites that have pages now indexed and listed displaying such a data. Terrible. This instills a potential for hazard that can easily be fixed.

So again. Make sure you're not leaving open the potential for sensitive information to be handed out in the wild. Errors happen whether we like it or not. When we have the chance to avoid them, we should take it. This goes for Adobe ColdFusion in a similar sense too!

Happy coding.
