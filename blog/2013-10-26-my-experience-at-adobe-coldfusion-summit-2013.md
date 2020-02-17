---
title: My Experience At Adobe ColdFusion Summit 2013
path: /my-experience-at-adobe-coldfusion-summit-2013
date: 2013-10-26
summary: My Experience At Adobe ColdFusion Summit 2013
tags: ["ColdFusion", "ColdFusion Summit"]
---

So I'm sitting at the airport waiting for my first of two flights back to Florida to come in. I'm one of those people who has to be punctual as much as possible; and this was no difference. Punctual in my book means early. Sometimes real early. Most of the time. In doing so, I am now holding 2hrs before my flight arrives. But hey, that's time to review the ColdFusion Summit of 2013 right?

This could get lengthy so I warn you now.

## The Conference & Keynotes as a Whole. . .

I'm gonna skip some of the dull bits; although I must say the breakfasts & lunches provided were the best I've ever had at a conference. Not sure if that says anything haha!

This was my first ColdFusion conference ever! Needless to say, I had a wonderful time. There's something magical when you've spent years working in a industry, on a specific platform, learning from the greats from an online presence and then get to put physical faces on them. There's so much more life breathed into what you do, and enjoy, when you immerse yourself with others, who enjoy it as well, in a "real-life" perspective. It just reinforces the truly beautiful experience as a whole.

I thought the event was well formed. Each day flowed for me with no issue; other than getting lost trying to find registration on Wednesday. Up early, hit the convention area, stuff my face, go to work! The session rooms were ideal; although the first day they were crazy cold. By day 2 however, Adobe had more or less fixed that issue. Kudos to those who kept up with all of the dynamic issues! For me, as long as I sat close to the front I could see the projectors. My eyesight is debatable from a distance sometimes. Especially for code. Looking at color-coded examples is murder from afar. My eyes were shot by the end of the day; both days.

I enjoyed both Thursday and Friday's keynotes. Day 1 was a cool experience for me as I got to see Ben Forta talk about ColdFusion, it's direction and just be awesome hehe. Ben is tall haha. While some might gawk at the typical blather of assuring and reassuring the life of CF and it's progression in sales etc., I appreciated hearing it from Ben; and Rakshith for that matter. Ben's display of going over the early days of CF and it's pitfalls along the way was enjoyable (even if I can only relate back to CF5). Between Ben and Rakshith there was a good amount of coverage as to where ColdFusion Splendor (CF11) will take us and what is to come from it.

#### Some key bits...

- Full CFScript support! (finally!) (e.g. `tag {attribute1="value" attribute2="value"};`)
- Member Functions! (e.g. `arrayLen()` can now be `array.len()` etc.)
- Revamp of CFQuery script support. - `queryExecute()`
- Improved JSON Serialization.
- Revamped PDF engine. - Better HTML/CSS conversion and PDF creation.
- The CFClient tag for mobile dev. - It sounds cool but I say meh.
- `<cfmail>` now has a encrypt attribute for encrypting emails.
- Implementation of the `Elvis Operator ?:`.
- Plus more things I'm sure I missed and/or have not been mentioned yet.

So some of the features I list above are some big steps for ColdFusion (even if they are OVERDUE!). Full tag support in script is a wonderful thing. Though the implementation is simply stripping off the `<cf` and `>`; it provides backwards support with Railo's implementation which is nice. I should also mention that you can write a tag like this in script as well - `tag(attribute="value");`. It's a welcomed feature in my book.

Member functions are also a nice addition that many of us have asked for. For a long, long time. Along with a nicer grasp to a OO style and such, it's cleaner to write in script, I think; which provides for better readability. There are those times when I have a few functions nested, an error is thrown complaining about a semicolon and I'm then scanning through brackets to see if there are any missing or there's too many. Maybe that's just something I do ;).

I'm not touching CFClient in the light of this article. There are great debates on Twitter about it and plenty of mixed impressions. Honestly, I think it's a cool concept but I don't see myself using it. To each their own. For anyone interested, Adam Tuttle [wrote a interesting blog post](http://fusiongrokker.com/post/cfsummit-2013-apk-teardown) on decompiling the Android version of the CFSummit app which was built using this tag and taking a peek under the hood.

CF Hour's 200th episode later that evening of day 1. . . Hilarious! That is all.

I enjoyed the keynote on day 2. It was an interesting presentation by Avi Rubin that really made you think twice about how secure something actually is. The presentation of ABE encryption and a project called CHARM was pretty cool as well. On a side note - The underlying encryption methods shown were written in C. I'm not surprised his examples were in Python. Some people felt he was trying to sell us on using Python and/or complaining that it wasn't "ColdFusion oriented". Really? Learn from example people. It was being presented from a real world scenario and could easily be self-portrayed in a CF mindset. Jason Dean did tweet that a Java implementation was being released down the road. That's cool. You can learn more about CHARM here.

## Sessions!

I loved the sessions I attended. Much of my choices were either a attempt to grow my knowledge or simply harden it and reinforce what I (think I) already know. I was delighted to attend some of the advanced topics and understand and grasp things. It's so reassuring when you listen to the pros go over something and you're already doing it by habit.

#### Adobe sessions by Vamseekrishna, Rupesh & Rakshith

These sessions included - Language Enhancements in ColdFusion, Preview: ColdFusion Splendor and Adobe ColdFusion - The Road Ahead

The sessions I attended presented by the Adobe guys were all related to upcoming bits from ColdFusion Splendor and ColdFusion Dazzle. Which I mention above and I have tweeted some highlights with pics on Twitter (follow me @tonyjunkes) as well; so I won't repeat myself here. They were good sessions. My one gripe is from Vamseekrishna's session. His slides were way too small and his code examples were even smaller. Someone in the audience helped him figure out how to make the font larger in CFBuilder but even that was no good. Plus I couldn't hear very well. Not so hot when a CF engineer can't get things to work. It flopped a bit but things happen I suppose. I gathered a lot of info from the sessions that involved the Adobe guys and I'm grateful. Rakshiths presentations were awesome. Even if he was talking a mile a minute; he covered a lot!

#### Java Integration - Dave Gallerizzo

I didn't know what to expect from this session. I thought I was either going to already know the drill or be wowed by something different. Java is not my forte but I know how to incorporate it to a good bit in ColdFusion (I use BCrypt and Twitter4j a lot). I was delightfully surprised to already know how to do pretty much everything presented. That doesn't make it a wasted session though. Dave is a ridiculously powerful speaker. He had my attention the entire time; and I left feeling more empowered on the knowledge than prior. He covered mostly how to load classes and JARS into ColdFusion's class path for CF9 and CF10 using their respected capabilities and how to call them. The one thing I had never used that he discussed was CFCProxy for calling CFC methods from Java. It's a neat concept.

#### Object-Oriented ColdFusion - Dan Wilson

I've been writing my apps in what I believe to be a OO oriented style for a while now. I've read a lot on the topic in relation with CF but wanted to dive into that next step and listen to someone go over it in a knowledgeable fashion. Dan did a great job going over the basic concepts and how it can really benefit you taking such an approach over the plain procedural practice; which can lead to a mess of spaghetti code. This was another session where I was happy to see I was doing something right on a general level.

#### Advanced Object-Oriented Programming in ColdFusion - Scott Strozz

This session followed right after Dan's which was perfect. Nothing like cramming an hour straight of Object-Oriented goodness back to back! I loved this session. Scott is a funny guy. He's also a smart CF guy. If you've never listened to CF Hour, do it. Scott and Dave Ferguson are a trip. Anyway. This session was really good. Scott covered things I didn't expect in the session. He was very thorough, in a easy to grasp manner, explaining a handful of OO patterns including MVC, DI/IOC, Services etc. What took me beyond my norm was he did a lot of examples with ORM (Object Relational Mapping). This is something I haven't used in a CF project. I know basics in Grails; but his use of explaining OO concepts along with ORM opened my mind up even more. So much that at the end of the day, I went to my room and started playing with ORM to convert my blog (which runs typical DB SQL) to ORM. All of that OOP made me ready to take that next step and I appreciate that.

#### Open Yourself to Closures - Adam Tuttle

Back when I first heard Adam was going to do a session on closures I knew immediately I was attending it. While I'm no pro with closures, I like the concept and I like using them when the opportunities seem capable/acceptable. He started out with going over closures v.s. callbacks and making the point that ColdFusion 10's new array and struct functions (i.e. arrayEach(), structEach()) are not actually closures on their own as they make use of a anonymous functions on their own. Something I feel has become misleading from how they were advertised originally. It's one of those poor manners in the CF world where someone believes they are doing something one way, all the way until they see the real thing and say "wtf?!". When he demoed the curry function, I understood from his explanation; but when he asked if anyone had any questions, there was silence (chuckle). He did a great job going over the real world of closures though. Showing examples from JavaScript and some of the converting projects in CF that take JS closure functions and port them to CF. My brain rolled over a few times; but I like that. It builds character.

#### Unit Testing - Kev McCabe

I don't unit test my code. I don't know how; but I've been wanting to learn so this session was ideal. This was another session where my head went heavy while trying to take it all in. Kev went over the general understanding and definition behind it and the concepts; and before long he was digging into part of Ray's Blog.cfc and testing parts of the code and refactoring it in the process. This method of teaching is something I like very much. While I wasn't the one doing the hands on, watching Kev go through code that I could understand, run it against some tests and proceed to make them go green was just as good as hands on. This is another area where I'm going to dedicate some time and play with a few testing frameworks and start making it a regular thing.

#### Application Security Best Practices Part II - Jason Dean

Security means a lot to me and I'm sorry I missed Jason's part I of this session. This is one more thing where I know I am no pro; but want to try and comprehend as much as possible. This, after all, leads to implementation of better practices. In this session he covered a good amount on cryptography and encryption in ColdFusion, Session Hijacking, XSS (Cross Site Scripting) and OWASP's ESAPI project. Jason does a good job at making the point quite clear; which is important with security. He also brought up a lot of settings and features of ColdFusion that are not optimal for solid security practices by default and how to handle that. "Adobe, fix that!".

## In Closing. . .

If you're interested in checking out the slides from various presenters... (the link is long dead). If anything I tweeted about ColdFusion Dazzle sounded interesting (which it is!), look for Rakshith's slides on "Adobe ColdFusion - The Road Ahead". If they're not there, harass him on Twitter :)

There was one more length of sessions I could have done on Friday however, I was really exhausted the second day in. It's a 3hr change backwards in time for me and it caught up fast by the end of the day. I ended up going back to my room to recharge and went back down for the closeout of the event at 5pm.

All in all, I had an amazing time! I intend to make the effort to come back for round 2 in 2014. I want to extend my thanks to Adobe, the ColdFusion team and the others who played key roles in making CF Summit 2013 a thing. I am grateful. I learned so freakin' much that I now have a list ahead of me to try out, play with and implement! This conference has made me a better ColdFusion developer; a better web developer. Period. I got to meet so many people that share the same passion as I do and that makes me happy. Thank you!

So I started this at 6:30 a.m. PST and now it's 9:00 p.m. EST here in Florida. It was such a busy day of flying and now here I am publishing this over a few beers downtown with a crazy Halloween event going on outside the bar. :)

Cheers all.