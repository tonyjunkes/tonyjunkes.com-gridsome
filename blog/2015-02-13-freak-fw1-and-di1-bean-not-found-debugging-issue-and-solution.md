---
title: Freak FW/1 & DI/1 Bean Not Found Debugging Issue / Solution
path: /freak-fw1-and-di1-bean-not-found-debugging-issue-and-solution
date: 2015-02-13
summary: Freak FW/1 & DI/1 Bean Not Found Debugging Issue / Solution
tags: ["CFML", "FW1", "DI1"]
---

So earlier today I was working on the back end for this blog;Â which runs on FW/1 and DI/1. I ran intoÂ an issue that set me back for a hot minute; mostly because the error was quite vague in regardsÂ and I also didn't stress my debugging options as much as I could/should have (initially)!

I was making updates to a service object handled by DI/1 for use in a controllerÂ and at one point, all of a sudden, the page using the service bombed on me consistently. After catching the error, all I was getting back was "bean not found: MyService". The stack trace wasn't lending any hints either. This threw me a bit short-sighted as the CFC was definitely where it needed to be but for some reason DI/1 wasn't picking it up. I gave up for the time being.

Then the light bulb went off in my head. To help in getting to the bottom of the issue I dumped out the structure returned by `getBeanFactory().getBeanInfo()` and sure enough, I had a hint to the problem. In the list I expected to see the object "MyService" but what I found instead was "MyServicedomains". Domains being my singleton location for Domain Objects. I took a look in the directory it linked to and there it was, a copy of the CFC. At some freak point I accidentally dropped it into the wrong directory. Removing it immediately synced things back to normal.

Such a silly mistake but given the scenario, not as apparent I thought. So there ya go: Having issues with DI/1 picking up an object? Dump out the bean info and see if it's hooking it up somewhere else.