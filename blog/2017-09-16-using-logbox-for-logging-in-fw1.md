---
title: Using LogBox For Logging In FW/1
path: /using-logbox-for-logging-in-fw1
date: 2017-09-16
summary: Using LogBox For Logging In FW/1
tags: ["CFML","FW1","LogBox"]
---

I've seen talk of FW/1 and LogBox integration come up a few times on the [CFML Slack](https://cfml-slack.herokuapp.com/), so I figured I'd run through a way to get that set up in your application and start using it.

### What Is LogBox?

> [LogBox](https://www.gitbook.com/book/ortus/logbox-documentation/details) is a standalone enterprise ColdFusion (CFML) logging library designed to give you flexibility, simplicity, and power when logging or tracing is needed in your applications.

Based on [Log4j](https://logging.apache.org/log4j/2.x/), LogBox offers capabilities to log events and data in your application using various [appenders](https://logbox.ortusbooks.com/content/appender_properties/) such as writing to a file, database, and email formats.

We're just going to scratch the surface in this article but let's break this down...

## Getting Started

### Prerequisites

#### CommandBox

The examples will be making use of CommandBox to build out dependencies so you will want to have that [installed](https://ortus.gitbooks.io/commandbox-documentation/content/setup/installation.html).

#### Setting Up Dependencies In box.json

Start CommandBox from a terminal/command prompt, point it at your project and create a `box.json` for CommandBox to identify with.

```bash
#> box
#> cd /path/to/fw1-logbox-example
#> init fw1-logbox-example
```

Now define your dependencies in box.json. At a minimum, we want to add the settings below.

```json
{
    "name":"fw1-logbox-example",
    "dependencies":{
        "fw1":"be",
        "logbox":"x"
    },
    "installPaths":{
        "fw1":"framework",
        "logbox":"subsystems/logbox/"
    }
}
```

This will tell CommandBox that we need the latest develop version of FW/1 and latest stable version of LogBox. We're also specifying that we want FW/1 to go in `/framework` and LogBox to go in `/subsystems` directories.

> Due to the modular nature of subsystems, I personally prefer treating LogBox as a module in this case but, you can put it anywhere you'd like.

#### Framework One Structure

You'll need an FW/1 application structured out. We can call it `fw1-logbox-example`.

Here's the minimal requirements for the project:

```md
fw1-logbox-example
|- /conf
|- /controllers
|- /layouts
|- /subsystems
|- /views
|-- /main
|--- default.cfm
|--- error.cfm
|- Application.cfc
|- index.cfm
```

> Note: /conf is not an FW/1 requirement. We'll use it to house some settings for LogBox.

And the base for our FW/1 settings in `Application.cfc`:

```js
variables.framework = {
    defaultSection: "main",
    defaultItem: "default",
    error: "main.error",
    diEngine: "di1",
    subsystems: { }
};
```

We'll add more to the settings shortly.

#### Finalizing the Setup

For ease of defining paths to LogBox, we'll create an application mapping.

Add this to your `Application.cfc`:

```js
this.mappings = {
    "/logbox" = expandPath("./subsystems/logbox")
};
```

Now we'll tell DI/1 where LogBox and its components are and define some arguments for any constructor methods.

First, we're going to need to create a `LogBoxConfig.cfc` file with our custom settings to pass to LogBox. This will go in the `/conf` directory.

```js
component {
    void function configure() {
        logBox = {
            appenders = {
                // RollingFileAppender outputs to the logs/MYTESTLOG.log file
                myTestLog = {
                    class = "logbox.system.logging.appenders.RollingFileAppender",
                    levelMax = "WARN",
                    levelMin = "FATAL",
                    properties = {
                        filePath = "/logs",
                        autoExpand = true,
                        fileMaxSize = 3000,
                        fileMaxArchives = 5
                    }
                }      
            },
            root = { levelmax = "DEBUG", levelMin = "FATAL", appenders = "*" }
        };
    }
}
```

Above we set up our configuration file to log to a physical file (MYTESTLOG.log) that uses the `RollingFileAppender`. Here we can define where the file will go as well. You can read up more on the options in the [Configuring LogBox](https://logbox.ortusbooks.com/content/configuring_logbox/) section of the docs.

Now we will declare the LogBoxConfig.cfc as a bean in DI/1 and pass it to LogBox.

Add this block of code to the empty `variables.framework.subsystems` struct in `Application.cfc`.

```js
logbox: {
    diLocations: "/logbox",
    diConfig: {
        loadListener: function(di1) {
            di1.declare("LogBoxConfig").instanceOf("logbox.system.logging.config.LogBoxConfig")
               .withOverrides({ CFCConfigPath: "conf.LogBoxConfig" })
               .done()
               .declare("LogBox").instanceOf("logbox.system.logging.LogBox")
               .withOverrides({ config: di1.getBean("LogBoxConfig") });
        }
    }
}
```

So we create an instance of LogBox's base LogBoxConfig object and pass in the path to our own config file as a parameter. We then create an instance of LogBox which takes the newly declared config object as its constructor.

There's one last piece to add to our `Application.cfc`. Currently, we have only defined LogBox to be available from the LogBox subsystem bean factory. This means we would have to call an instance of it by doing `getBeanFactory("logbox").getBean("LogBox")` every time. For property injection to work across our application, we'll need to make it available to the parent bean factory.

In `variables.framework`, we'll add the following:

```js
diConfig: {
    loadListener: function(di1) {
        di1.declare("Logger").asValue(getBeanFactory("logbox").getBean("LogBox"));
    }
}
```

This declares a bean, called `Logger` (or whatever you'd like to call it), which holds an instance of the LogBox bean from the subsystem bean factory.

Now we can inject the bean into a controller or service and use it like so:

```js
// controllers.main
component accessors=true {
    property Logger;

    void function default(struct rc) {
        Logger.getLogger(this).fatal("Oh no an error!");
    }
}
```

In main.cfc, the Logger object uses `getLogger(this)` to get the logger we defined in LogBoxConfig and `fatal(message)` is used to used to log a "fatal" message. You can refer to [Using a Logger Object](https://logbox.ortusbooks.com/content/using_a_logger_object/) in the docs for more info on the various message methods.

## Putting It All Together

Now that we have our application settings put together, we can use CommandBox to pull in our dependencies and start a server instance.

From the project directory run:

```
#> install && start
```

This will pull in Framework One and LogBox from [ForgeBox](https://www.forgebox.io/), start an instance of Lucee and open the application in a browser.

Behind the scenes, the main controller was requested and executed the Logger methods for writing to the log file. If we check in [project root]/logs we'll see `MYTESTLOG.log` was generated and contains the message passed to `fatal()`.

## What's Next?

With a working instance of LogBox that you can pass around to your controllers and services, the logging options are endless.

Refer to the [documentation](https://logbox.ortusbooks.com/content/) on how to further customize the configuration and utilize the object functions for logging.

To see the above setup in action, you can download an [example project from GitHub](https://github.com/tonyjunkes/fw1-logbox-example).

Cheers.
