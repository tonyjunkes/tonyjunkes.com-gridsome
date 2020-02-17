---
title: CommandBox, Rewrite Woes & How to Debug Them
path: /commandbox-rewrite-woes-and-how-to-debug-them
date: 2017-09-15
summary: CommandBox, Rewrite Woes & How to Debug Them
tags: ["CommandBox","URL Rewrites","Debugging"]
---

Over the last day and a half, I've been plagued by what seemed an oddity when working on a logging demo in CommandBox. It had me thrown to the point of thinking I came across some strange bug. So I wanted to touch on how to debug the network requests and responses going on behind the scenes of a CommandBox server instance.

#### What was happening?

Well, when I fired up my application in CommandBox and went to check for a log entry, I was literally seeing double. Two identical log entries and no idea how, or why, it was happening. At this stage, I was running an app using FW/1 but after checking with Sean Corfield on the CFML Slack, we were able to confirm there were two requests happening but not due to FW/1. Off to the box-products channel I went, to query Brad Wood. Brad, as always, was quite helpful (cheers Brad) and had an answer for me right away along with a handy tip on how he came to the result.

#### So what really happened?

Apparently, the browser I was using (Chrome) was sending a request for a `favicon.ico` file that, in this case, clearly did not exist. The bad request, it seems, is still caught and matched to a rewrite which, in turn, tries to make a request to `/index.cfm/favicon.ico`. This obviously doesn't exist either but the request still makes it far enough to trigger the logs I had in place. This resulted in an extra request than I expected. This ultimately was due to me having `"rewrites":{"enable":true}` in my `server.json`. I assume the rewrite that catches this is something set up in Undertow/Tuckey Rewrite Filter for the default rewriting available from CommandBox. I'm definitely not sure of this, however.

> Apparently it's a known issue that Chrome makes this request every time. I'm not sure about other browsers.

#### How did we come to this?

When firing up a server instance in CommandBox, you can make debugging info available right in the console.

```bash
#> server start --debug --console
```

This made it possible to see the requests being made and how the container/rewrite filter was processing/directing those requests.

#### How can you get around this?

The short answer is to filter out the request with a rewrite but, one way to deal with this is to include a rewrite that matches `favicon.ico` and return a 403/404/something.

In CommandBox, this can be done by adding a rule to a `urlrewrite.xml` for the Tuckey Rewrite Filter to pick up.

```xml
<?xml version="1.0" encoding="utf-8"?>
<urlrewrite>
    <rule>
        <name>Intercept favicon.ico request</name>
        <from>^/favicon.ico$</from>
        <set type="status">403</set>
        <to>null</to>
    </rule>
</urlrewrite>
```

And including it in your server.json.

```json
"rewrites":{
    "enable":true,
    "config":"urlrewrite.xml"
}
```


That's all for now. I thought the debugging features were pretty slick and super helpful in figuring out something that, while minor, was driving me crazy.

Cheers.
