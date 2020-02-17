---
title: Working With FW/1 & QB - Steps to Integrate Using Subsystems
path: /working-with-fw1-and-qb
date: 2017-08-22
summary: Working With FW/1 & QB
tags: ["CFML","FW1","QB","SQL"]
---

Today I wanted to go over a library that I think is pretty awesome and echoes capabilities found in many other modern languages: QB.

[QB](https://www.forgebox.io/view/qb) is a query builder DSL written by [Eric Peterson](https://github.com/elpete), who's been putting out some [really cool modular libraries](https://www.forgebox.io/user/elpete) for the CFML community.

With QB, you can:

- Quickly scaffold simple queries
- Make complex, out-of-order queries possible
- Abstract away differences between database engines

The syntax uses a builder pattern that makes writing queries more readable and easy to understand when glancing through code.

As simple as:

```js
query.from("Posts").get();
```

And complex as:

```js
query.from("Posts")
     .whereNotNull("PublishDate")
     .whereIn("AuthorID", [1,2,3])
     .get();
```

QB is written to integrate with ColdBox as a module but, in this post, I'll go over how to integrate it as a subsystem in an FW/1 application and mimic the WireBox dependencies with FW/1's baked in dependency injection library, DI/1. While FW/1 and ColdBox have many differences in feature set, their convention to MVC patterns and syntax is still quite relatable.

> If you're ever glancing through ForgeBox and see a standalone library that's built to integrate with ColdBox, integrating it in your FW/1 app might (emphasis on might!) be easier than you think; as I'll demonstrate below.

## Getting Started
---

### Prerequisites

#### CommandBox

The examples will be making use of CommandBox to build out dependencies so you will want to have that [installed](https://ortus.gitbooks.io/commandbox-documentation/content/setup/installation.html).

#### Framework One Structure

_I'm going to assume you have a fair grasp of FW/1, configuring DI/1 in an FW/1 application and basic knowledge of how to use FW/1's subsystems feature_. If not, no worries. You'll still be able to follow along and I'll do my best at explaining and pointing out documentation when I can.

In my examples I'll be using Lucee and an H2 database using MySQL dialect, but you can use whatever engine/db combo you'd like. H2 is an in memory database that is easy to set up in a Lucee Application.cfc without having to touch the admin settings. _Setting up the H2 driver in Adobe ColdFusion may differ as I've never tried._

You'll want a basic FW/1 application fleshed out. Create a project directory and name it fw1-qb-example (or whatever you'd like). Next, build out a folder structure that follows the standard conventions like so...

```md
fw1-qb-example
|- /controllers
|- /layouts
|- /model
|-- /data
|- /subsystems
|- /views
|-- /main
|--- default.cfm
|--- error.cfm
|- Application.cfc
|- index.cfm
```

> Some convention based directories will not be used in my examples such as controllers &amp; layouts.

Your framework settings in `Application.cfc` should capture something like this to start:

```js
variables.framework = {
    defaultSection: "main",
    defaultItem: "default",
    error: "main.error",
    diEngine: "di1",
    diLocations: "/model",
    subsystems: { },
    trace: true,
    reloadApplicationOnEveryRequest: true
};
```

#### If Running Lucee: Setup For H2 Database

Create a directory in `/model` called `data`. This is where H2 will store it's temp files. Based on the structure above, all you need to do is add a few lines of code to your `Application.cfc`.

```js
this.datasource = "qb_test";
this.datasources[this.datasource] = {
    class: "org.h2.Driver",
    connectionString: "jdbc:h2:#expandPath("/model/data/" & this.datasource)#;MODE=MySQL"
};
```

> Note: H2 supports a number of mainstream SQL dialects.

### Putting It All Together

So far, you should have locked down getting CommandBox and a basic FW/1 structure ready. You may have noticed I didn't include FW/1's actual files in the breakdown. That's because we're going to pull it in as a dependency along with QB. Let's go ahead and do that now.

#### Installing Dependencies With box.json

First, we need to fire up CommandBox from a terminal/command prompt, point it at our project and create a project file (`box.json`) for CommandBox to identify with.

```bash
#> box
#> cd /path/to/fw1-qb-example
#> init fw1-qb-example
```

`init` creates our `box.json` to define dependencies; among other bits, we won't cover today. The file will have pre-generated settings but we really just need something that looks like this:

```json
{
    "name":"fw1-qb-example",
    "dependencies":{
        "fw1":"be",
        "qb":"x"
    },
    "installPaths":{
        "fw1":"framework",
        "qb":"subsystems/qb/"
    }
}
```

We've defined 2 key things here:

- `dependencies`: The libraries we want to get from [ForgeBox](https://www.forgebox.io/). `be` refers to the latest "bleeding edge" version (FW/1 is good about reasonably stable dev releases) and `x` means to get the latest stable release.
- `installPaths`: Where we want those libraries to be physically placed. FW/1 will go in /framework and QB will go in /subsystems/qb.

Once our dependencies are fleshed out, we can go ahead and tell CommandBox to install them.

```bash
#> install
```

#### Final Settings

At this point, we now have a functioning FW/1 application with QB set up as a subsystem on a very minimal level. It's time to wire up QB and its components in DI/1. This way we can access it directly in FW/1 and also satisfy its own dependencies.

For ease of defining paths to QB, we'll create an application mapping.

Add this to your `Application.cfc`:

```js
this.mappings = {
    "/qb" = expandPath("./subsystems/qb")
};
```

Next, we will tell DI/1 where QB and its components are and define some arguments for a constructor method.

Add this block of code to the empty `variables.framework.subsystems` struct in `Application.cfc`.

```js
qb.diLocations: "/qb/models",
qb.diConfig: {
    loadListener: function(di1) {
        di1.declare("BaseGrammar").instanceOf("qb.models.Grammars.Grammar").done()
           .declare("MySQLGrammar").instanceOf("qb.models.Grammars.MySQLGrammar").done()
           .declare("QueryUtils").instanceOf("qb.models.Query.QueryUtils").done()
           .declare("QueryBuilder").instanceOf("qb.models.Query.QueryBuilder")
           .withOverrides({
                grammar: di1.getBean("MySQLGrammar"),
                utils: di1.getBean("QueryUtils"),
                returnFormat: "array"
           });
    }
}
```

If you have a look in the `ModuleConfig.cfc` in the QB directory, you'll see some declarations set up with WireBox. The above code is essentially the equivalent mimicked in FW/1. Keep in mind, there's more than one way to accomplish what we have above so you may be familiar with a different approach.

So what's going on here?

`qb.diLocations: "/qb/models"` tells DI/1 to create a subsystem bean factory and gather objects in /qb/models based on the default conventions.

`qb.diConfig` is more involved...

We define a [load listener](http://framework-one.github.io/documentation/using-di-one.html#using-load-listeners) with a closure that takes the DI/1 object as an argument. This is for defining our bean factory settings. In the function block, we use DI/1's convenient builder syntax to declare individual bean objects of QB's components. The `declare()` method is used to define an "alias" to the component. This is useful for having DI/1 automagically satisfy an object's constructor arguments. The last declaration of "QueryBuilder" calls `withOverrides()` to pass in specific arguments to the the object's constructor, or init method.

> Note: The declaration of MySQLGrammar. This is specific support in QB for the MySQL dialect. There are other options available. See the /qb/models/Query/Grammars folder.

Now that we've programmatically squared away our component requirements, we're ready to fire up a server and actually do things!

From CommandBox we just need to enter `start` and a server will start up and open a browser with our application.

## Examples
---

To use QB from DI/1, we can call on it like so:

```js
builder = getBeanFactory("qb").getBean("QueryBuilder");
```

Let's assume some simple scenarios...

Given a `Posts` table, return all posts.

```js
// Returns an array of structs
posts = builder.from("Posts").get();
```

Given a `Posts` table, return all posts that are not drafts.

```js
posts = builder.from("Posts").where("IsDraft", "=", 0).get();
```

Given a `Posts` table, return post by ID.

```js
posts = builder.from("Posts").where("ID", "=", 5).get();
```

Given a `Posts` table, return all posts that include "CFML" in the title.

```js
posts = builder.from("Posts").whereLike("Title", "%CFML%").get();
```

Given a `Posts` &amp; `Authors` table, return all posts by that author.

```js
posts = builder.from("Posts")
               .join("Authors", "Authors.ID", "=", "Posts.AuthorID")
               .get();
```

Given a `Posts` &amp; `Authors` table, return all posts by that author within a date range.

```js
posts = builder.from("Posts")
               .join("Authors", "Authors.ID", "=", "Posts.AuthorID")
               .whereBetween("PublishDate", createDate("2017", "8", "1"), now())
               .get();
```

## What's Next?

That was just a few examples to scratch the surface of what QB is capable of. You can refer to the [Official QB Docs](https://elpete.gitbooks.io/qb/content/) for more examples. You can always glance through the source code to see what has been implemented as well.

Kudos to Eric Peterson for creating such a cool library.

If you work with FW/1 apps and haven't taken the plunge into incorporating CommandBox / ForgeBox with your typical workflow, I hope this helps get the wheels turning.

I've also put together an [example application on GitHub](https://github.com/tonyjunkes/fw1-qb-example) based on this post.

Happy coding!
