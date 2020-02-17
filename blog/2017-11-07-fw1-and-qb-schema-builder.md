---
title: FW/1 & QB Schema Builder
path: /fw1-and-qb-schema-builder
date: 2017-11-07
summary: FW/1 & QB Schema Builder
tags: ["CFML","FW1","QB","SQL"]
---

Not too long ago I wrote an article about [working with FW/1 and QB](http://tonyjunkes.com/blog/working-with-fw1-and-qb/); a query builder DSL written by [Eric Peterson](https://github.com/elpete).

In this post, I wanted to touch on a new feature and component released in the latest version of QB, [Schema Builder](https://elpete.gitbooks.io/qb/content/schema/), and how to make use of it in an FW/1 application.

Schema Builder presents the ability to build out your database structure using the same awesome builder/chaining format as the Query Builder object. This allows us to create table layouts from the ground up in straight CFML.

Here's an example from the docs:

```js
schema.create("users", function(table) {
    table.increments("id");
    table.string("email");
    table.string("password");
    table.timestamp("created_date").nullable();
    table.timestamp("modified_date").nullable();
});
```

## Getting Started / Prerequisites
---

In my previous post, I went over how to setup and use QB in an FW/1 application. If you're unfamiliar with the setup, [I highly recommend glancing through its content](http://tonyjunkes.com/blog/working-with-fw1-and-qb/) as I'll only be adding on to those code examples here.

So to tie everything together you'll want to have CommandBox installed, a basic FW/1 application structure using subsystems, a box.json for defining dependencies like FW/1 and QB (all explained in the linked post above) and a mapping in Application.cfc to the path where QB is kept.

## Wiring Things Up With DI/1
---

With a functioning FW/1 application and QB set up as a subsystem, It's time to wire up QB and its components in DI/1.

Within the `variables.framework.subsystems` struct in `Application.cfc`, we'll define the QB components and their dependencies while also including the `SchemaBuilder` component.

```js
qb.diLocations: ["/models"],
qb.diConfig: {
    loadListener: function(di1) {
        di1.declare("BaseGrammar").instanceOf("qb.models.Grammars.BaseGrammar").done()
           .declare("MySQLGrammar").instanceOf("qb.models.Grammars.MySQLGrammar").done()
           .declare("QueryUtils").instanceOf("qb.models.Query.QueryUtils").done()
           .declare("QueryBuilder").instanceOf("qb.models.Query.QueryBuilder")
           .withOverrides({
                grammar: di1.getBean("MySQLGrammar"),
                utils: di1.getBean("QueryUtils"),
                returnFormat: "array"
           }).done()
           .declare("SchemaBuilder").instanceOf("qb.models.Schema.SchemaBuilder")
           .asTransient()
           .withOverrides({
                grammar: di1.getBean("MySQLGrammar")
           });
    }
}
```

#### Breaking It Down

`qb.diLocations: "/models"` tells DI/1 to create a subsystem bean factory and gather objects in /qb/models based on the default conventions.

In `qb.diConfig` we define a [load listener](http://framework-one.github.io/documentation/using-di-one.html#using-load-listeners) with a closure that takes the DI/1 object as an argument. This is for defining our bean factory settings. In the function block, we use DI/1's builder syntax to declare individual bean objects of QB's components. The `declare()` method is used to define an "alias" to the component. This is useful for having DI/1 automagically satisfy an object's constructor arguments. The declaration of "QueryBuilder" calls `withOverrides()` to pass in specific arguments to the object's constructor, or init method. Then we define "SchemaBuilder" as a transient object which takes the same grammar object (MySQLGrammar) as QueryBuilder.

> Note: The declaration of MySQLGrammar. This is specific support in QB for the MySQL dialect. There are other options available. See the /qb/models/Grammars folder.

Now we just need to start our application from CommandBox to have a working environment.

## Examples
---

#### Usage

To use the QueryBuilder from DI/1, we can call on it like so:

```js
QueryBuilder = getBeanFactory("qb").getBean("QueryBuilder");
```

To use SchemaBuilder from DI/1, we call it the same as QueryBuilder:

```js
SchemaBuilder = getBeanFactory("qb").getBean("SchemaBuilder");
```

#### Creating Tables

Let's build out a posts and authors table.

```js
// Authors
SchemaBuilder.create("authors", function(table) {
    table.increments("UserID");
    table.string("FirstName", 150);
    table.string("LastName", 150);
    table.string("Username");
    table.string("Password", 60);
    table.timestamp("CreateDate");
    table.timestamp("ModifyDate");
});

// Posts
SchemaBuilder.create("posts", function(table) {
    table.increments("PostID");
    table.string("Slug");
    table.string("Title");
    table.longText("Body");
    table.integer("ViewCount");
    table.string("Keywords");
    table.string("Description");
    table.timestamp("CreateDate");
    table.timestamp("ModifyDate");
    table.timestamp("PublishDate");
    table.bit("IsDraft");
    table.bit("Active");
    table.unsignedInteger("UserID").references("UserID").onTable("authors");
});
```

The above examples demonstrate scaffolding out columns of various data types as well as defining incrementing IDs and constraints on another table.

#### Checking If A Table Exists

```js
if (!SchemaBuilder.hasTable("posts")) {
    SchemaBuilder.create("posts", function(table) {
        // scaffold columns...
    });
}
```

#### Dropping Tables

```js
// Drop
SchemaBuilder.drop("posts");

// Drop if exists
SchemaBuilder.dropIfExists("authors");
```

#### Altering Columns

```js
// Add
SchemaBuilder.alter("authors", function(table) {
    table.addColumn(table.boolean("active"));
});

// Drop
SchemaBuilder.alter("authors", function(table) {
    table.dropColumn("active");
});

// Modify
SchemaBuilder.alter("authors", function(table) {
    table.modifyColumn("name", table.string("firstname"));
});

// Rename
SchemaBuilder.alter("authors", function(table) {
    table.renameColumn("name", table.string("firstname"));
});
```

#### Populating

We can then use QueryBuilder to populate the created tables.

```js
// Author
QueryBuilder.from("authors").insert(
    values = {
        firstname: "John",
        lastname: "Doe",
        username: "jdoe",
        password: "tvBvOpODv4BiPGwCcPFeenYIVFis6LuDgqX",
        createdate: now(),
        modifydate: now()
    }
);

// Post
QueryBuilder.from("posts").insert(
    values = {
        slug: "test",
        title: "Test",
        body: "This is a test.",
        viewcount: 0,
        keywords: "",
        description: "",
        createdate: now(),
        modifydate: now(),
        publishdate: now(),
        isdraft: 0,
        active: 1,
        userid: 1
    }
);
```

#### Retrieving Data

```js
QueryBuilder.from("Posts")
            .join("Authors", "Authors.ID", "=", "Posts.AuthorID")
            .get();
```

## What's Next?

Refer to the [Official QB Docs](https://elpete.gitbooks.io/qb/) for more examples.

To further see FW/1 & QB in action, you can check out my [example application on GitHub](https://github.com/tonyjunkes/fw1-qb-example).

Cheers.
