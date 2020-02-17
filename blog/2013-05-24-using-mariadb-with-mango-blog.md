---
title: Using MariaDB with Mango Blog
path: /using-mariadb-with-mango-blog
date: 2013-05-24
summary: Using MariaDB with Mango Blog
tags: ["ColdFusion", "MariaDB", "Mango Blog"]
---

In one of my previous posts I went over [connecting ColdFusion to MariaDB](http://www.tonyjunkes.com/blog/connecting-coldfusion-to-mariadb). This was my first hands on with MariaDB. Once I had things talking I went to test using MariaDB as the database to connect with Mango Blog.

Initially, when switching, I was able to view the main pages - posts and the other content. I could log in to the admin as well; but upon trying to create a post or edit content, I would get a ColdFusion error regarding invalid MySQL syntax. Looking back on how I set up MariaDB to CF, I had used the `other` option when adding a data source in the CF Admin. Since Mango Blog's setup allows for using MSSQL and MySQL, there was most likely a hang-up in identifying what DBMS was actually being used. I'm not entirely sure as I have not pursued this error fully.

The solution that works actually came in part to a comment made on the above referenced post. Since MariaDB is a fork of MySQL, when setting up a MariaDB data source in the CF Admin, you don't have to go the "other" route. Instead you can simply use the `MySQL 4/5` option and, in the "Server" field, specify the URL and port to MariaDB. This will hook things up more or less the same as using the "other" option but the key difference here is that the underlying reference to the DBMS will still come off as MySQL from CF.

When connected in this way you can easily choose that data source as a MySQL database to use with Mango Blog; but it will in fact be using the MariaDB database. So far in my tests everything runs and functions as expected. Obviously there is the possibility for this to fail in the future as features / inner-workings of MariaDB shift from MySQL. Of course this was more experimental than anything.

Cheers.