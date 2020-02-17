---
title: Migrating Legacy FW/1 Subsystems
path: /migrating-legacy-fw1-subsystems
date: 2018-03-17
summary: Migrating Legacy FW/1 Subsystems
tags: ["CFML","FW1"]
---

Today I'm going to go over how to upgrade the modular capabilities of your crusty old (_read: beautifully written_) FW/1 application. While you may have been keeping the framework libraries up to date, perhaps you didn't take the plunge and move your "legacy" architecture over. That'll be our target. Note: If you're still running on top of anything pre-v3.5, this will not apply to you; however, I highly recommend you upgrade to the latest and greatest. One minor->major version release at a time as they tend to introduce breaking changes without actually breaking your code - yet.

Anyway...

In 2015, the release of [FW/1 3.5](http://framework-one.github.io/blog/2015/10/21/fw1-3-5-0-released/) brought us `Subsystems 2.0`.

> "A new, more streamlined way to add subsystems to an existing FW/1 application".

This was a big deal, to me. As a heavy user of subsystems and a fan of modularity, this not only made the architecture more organized but it also made setup super simple and much more user-friendly. /salespitch ;)

## The Old (Legacy) Way

The original way of working with subsystems in FW/1 involved essentially everything being a subsystem; including a common location for shared layouts and views.

Consider this application structure:

```md
legacy_project
|- /admin
|-- /controllers
|--- main.cfc
|-- /layouts
|--- default.cfm
|-- /views
|--- /main
|---- default.cfm
|- /common
|-- /layouts
|--- default.cfm
|- /framework
|- /home
|-- /controllers
|--- main.cfc
|-- /layouts
|--- default.cfm
|-- /views
|--- /main
|---- default.cfm
|---- error.cfm
|- Application.cfc
|- index.cfm
```

So what do we have here?

- `/home` is our base (core) of our application. In an FW/1 application not using subsystems, this would potentially be the meat of our app. In this case, it's also being treated as a subsystem. It contains all of the usual base app items - controllers, views, layouts etc.
- `/admin` is a subsystem within our application. Any other subsystem can link to it, borrow from it or whatever purpose it built from it. The structure will mimic `/home` but it can act on its own within our application.
- `/common` is the "shared" source for layouts and views among all subsystems in the application. It is also treated as a subsystem.

Everything else should be relatively self-explanatory as it follows the standard conventions of any normal FW/1 application.

Now let's look at the configuration settings in `Application.cfc` that allow all of this to come together...

```js
component extends="framework.one" {
    // Application settings (this scope)...

    // FW/1 settings
    variables.framework = {
        action = "action",
        defaultSection = "main",
        defaultItem = "default",
        usingSubsystems = true,
        defaultSubsystem = "home",
        siteWideLayoutSubsystem = "common",
        error = "home.error",
        generateSES = false,
        SESOmitIndex = false,
        diEngine = "di1",
        diComponent = "framework.ioc"
    };

    // Lifecycle functions go here...
}
```

There are 3 main config settings that will make the above structure function.

- `usingSubsystems` [boolean] : As the name suggests, this is the flag that tells FW/1 whether or not your application is using subsystems.
- `defaultSubsystem` [string] : This defines the "core" of your application. It's often a starting point for a shared model or "home page".
- `siteWideLayoutSubsystem` [string] : The name of the folder that houses shared layouts/views to be used as defaults for any subsystem.

With this understanding, let's move on to switching to the current, modern way of using subsystems.

## The New [2.0] Way

Let's talk migration now. I'd like to point out at the start that it is not a difficult process. If anything, it requires you to make some architectural decisions as this method allows you to follow a "core vs modular" setup. Mind you, it's not completely required and I'll go into more detail shortly.

### Framework Settings

We get to start by deleting some code. Previously you would have to tell FW/1 that you wanted to use subsystems, specify a base and possibly name the shared layout. This is no longer required. Instead, by convention, FW/1 will look for a `subsystems` directory containing all of your subsystems. The only setting you might consider using is `subsystemsFolder` which takes a string value representing the name of the directory you wish to house your subsystems in.

```js
component extends="framework.one" {
    // Application settings (this scope)...

    // FW/1 settings
    variables.framework = {
        action = 'action',
        defaultSection = 'main',
        defaultItem = 'default',
        subsystemsFolder: "subsystems",
        error = 'main.error',
        generateSES = false,
        SESOmitIndex = false,
        diEngine = 'di1',
        diComponent = 'framework.ioc'
    };

    // Lifecycle functions go here...
}
```

### The Structure

So there are two ways to go about migrating the physical structure. It's worth noting that this structure is essentially a normal FW/1 app only you have a subsystems folder added to the mix. The base of your application code can still consist of controllers, views layouts etc. as your core/shared/home page of the application.

So we can:

**a.)** Completely build our application from a modular perspective and put every previous subsystem in the `subsystems` directory and put our shared layouts and views from the `siteWideLayoutSubsystem` folder into the base `/layouts` and `/views`.

```md
current_project_1
|- /layouts
|-- default.cfm
|- /controllers
|- /views
|- /framework
|- /subsystems
|-- /admin
|--- /controllers
|---- main.cfc
|--- /layouts
|---- default.cfm
|--- /views
|---- /main
|----- default.cfm
|-- /home
|--- /controllers
|---- main.cfc
|--- /layouts
|---- default.cfm
|--- /views
|---- /main
|----- default.cfm
|----- error.cfm
|- Application.cfc
|- index.cfm
```

**b.)** Everything that's in the `defaultSubsystem` goes into the base application folders (controllers, layouts, views etc.) and every other subsystem goes into the `subsystems` folder.

```md
current_project_2
|- /controllers
|-- main.cfc
|- /layouts
|-- default.cfm
|- /views
|-- /main
|--- default.cfm
|--- error.cfm
|- /framework
|- /subsystems
|-- /admin
|--- /controllers
|---- main.cfc
|--- /layouts
|---- default.cfm
|--- /views
|---- /main
|----- default.cfm
|- Application.cfc
|- index.cfm
```

## Done.

That's all there is to it. Now depending on how your subsystems may rely on/borrow from each other, you may need to decide what goes in the base of your application and what remains an actual subsystem. The best rule of thumb is all shared/core functionality goes in the base. All self-contained pieces remain in their respective subsystem. For the most part, everything should "just work" by moving it all into the `subsystems` folder.

Hopefully, I've explained everything in a sensible way. I've put together a [repository on GitHub](https://github.com/tonyjunkes/fw1-migrate-legacy-subsystems-example) containing 2 separate applications you can run with CommandBox or in your own local setup to see them in action along with the folder structure. When in doubt, always refer to the [official documentation](http://framework-one.github.io/documentation/4.1/using-subsystems.html#subsystems-10-vs-20).

Cheers.
