---
title: FW/1 Alternative Application Structure
path: /fw1-alternative-application-structure
date: 2017-11-18
summary: FW/1 Alternative Application Structure
tags: ["CFML","FW1"]
---

At Adobe ColdFusion Summit 2017, Masha Edelen presented her talk - [Power Of Simplicity In FW/1](https://www.slideshare.net/MashaEdelen/power-of-simplicity-in-fw1). In her presentation, she mentioned setting up the application structure by including the actual framework code in the webroot; explaining that in order to move that code out, you would need a global server mapping. So I figured I'd go over FW/1's "[alternative application structure](http://framework-one.github.io/documentation/4.1/developing-applications.html#alternative-application-structure)" and share how I like to implement it.

## What Is It?

Basically, this structure allows you to abstract the FW/1 lifecycle methods and configuration settings into a separate component. Essentially leaving your Application.cfc in a more "traditional" state aside from the calls needed for FW/1.

From the docs:

> It creates the FW/1 instance explicitly on each request and delegates the various lifecycle methods to FW/1. The framework configuration structure must be passed to the FW/1 constructor, instead of being set in variables scope.

## The Benefits

#### Application Mappings & Best Practices

A best-practice-rule-of-thumb is to keep your application code out of the webroot. This excludes Application.cfc, index.cfm and resource files directly used with the HTML UI.

Assuming you are keeping your application code out of the webroot (_which you should!_), you would need a server level mapping set in the administrator to make FW/1 available to the project. Lucee users can avoid this by extending a path in Application.cfc like `extends="../path/to/fw1"`; however this is not the case with ColdFusion.

The alternative structure allows you to avoid having to set a server level mapping from the administrator or utilize engine specific features. Instead, you can define an application mapping in your Application.cfc to point to the component that extends FW/1.

## Example Application

> You can find [example files](https://github.com/framework-one/fw1/tree/develop/framework) that relate to the docs in the FW/1 GitHub repo, but I'm going to be laying things out a little differently.

#### FW/1 Structure

All we need to start is a typical structure for FW/1 and I'll touch on the specific code after. In my example, I use `/src` for the application source code and `/app` as the webroot.

```md
fw1-lifecycle-example
|- /app
|-- Application.cfc
|-- index.cfm
|- /src
|-- /controllers
|-- /layouts
|-- /views
|--- /main
|---- default.cfm
|---- error.cfm
```

#### CommandBox All The Things

I'm going to use CommandBox to run my example so I'll create, in the project root, a `box.json` to get FW/1 and a `server.json` to specify the webroot (/app). This way all I'll need to do is run `box install && start` and we're off to the races.

_box.json_
```json
{
    "name":"fw1-lifecycle-example",
    "dependencies":{
        "fw1":"be"
    },
    "installPaths":{
        "fw1":"src\\framework"
    }
}
```

_server.json_
```json
{
    "name":"fw1-lifecycle-example",
    "web":{
        "webroot":"app",
        "rewrites":{
            "enable":true
        }
    }
}
```
#### The Lifecycle

There are 2 key parts to tying this FW/1 application structure together. The Application.cfc for creating the FW/1 instance and what I've decided to call the "Bootstrap" component that extends FW/1 to set framework settings and call its lifecycle methods.

##### The "Bootstrap" Component

This is where we will define FW/1 related variables and its lifecycle methods.

2 things to note:

- You can name this CFC anything you want.
- You can place this file anywhere you want _out of the webroot_.

What I like to do is create a `/conf` folder and name the component `Bootstrap.cfc`. I'm a fan of extending concepts I've used in Grails into my CF stuff.

Let's see what that might look like.

```js
component displayname="FW/1 Lifecycle Bootstrap" extends="framework.one"
    output=false
{
    // FW/1 settings
    variables.framework = {
        base: "/src",
        defaultSection: "main",
        defaultItem: "default",
        error: "main.error",
        diEngine: "di1",
        routes: [
            { "/" = "/main/default" }
        ]
    };

    public void function setupApplication() { }

    public void function setupEnvironment(string env) { }

    public void function setupSession() { }

    public void function setupRequest() { }

    public void function setupResponse(struct rc) { }

    public void function setupSubsystem(string module) { }

    public void function setupView(struct rc) { }

    public string function onMissingView(struct rc) {
        return "Error 404 - Page not found.";
    }
}
```

The content here should be familiar. It's the usual configurations for FW/1. Only now it is being placed in an external component. Also, notice that this is where FW/1 will be extended. In this example, the `/framework` folder is installed in `/src` where `/conf/Bootstrap.cfc` also lives. By adding an application mapping in `Application.cfc`, we can easily extend it as expected. No trip to the administrator.

##### Application.cfc

In the code example below, you'll see some familiar bits relating to the `this` scope and where we define necessary mappings to gain access to the application source code.

Moving down the line, we come to the `_get_framework_one()` method which is where we set an instance of the Bootstrap component to a request variable that FW/1 will work with.

> For simple framework config, instead of setting the variables in Bootstrap.cfc, you can pass a struct of those framework settings into the instance being set in `request._framework_one`.

The remaining code consists of the standard Application.cfc lifecycle methods; but in each one, we return FW/1's respective method calls from the "Bootstrap" instance.

```js
component
    output=false
{
    this.name = hash(getBaseTemplatePath());
    this.applicationTimeout = createTimeSpan(0, 2, 0, 0);
    this.sessionManagement = true;
    this.sessionTimeout = createTimeSpan(0, 0, 30, 0);
    this.mappings = {
        "/src" = expandPath("../src"),
        "/framework" = expandPath("../src/framework")
    };

    public Bootstrap function _get_framework_one() {
        if (!request.keyExists("_framework_one")) {
            request._framework_one = new src.conf.Bootstrap();
        }
        return request._framework_one;
    }

    public void function onApplicationStart() {
        return _get_framework_one().onApplicationStart();
    }

    public void function onError(exception, event) {
        return _get_framework_one().onError(exception, event);
    }

    public void function onRequest(targetPath) {
        return _get_framework_one().onRequest(targetPath);
    }

    public void function onRequestEnd() {
        return _get_framework_one().onRequestEnd();
    }

    public void function onRequestStart(targetPath) {
        return _get_framework_one().onRequestStart(targetPath);
    }

    public void function onSessionStart() {
        return _get_framework_one().onSessionStart();
    }
}
```

## Next?

All of the hard work is done! You can now flesh out the remains of an application with controllers, layouts, views etc. as you normally would. In this case, placing those files in the `/src` folder.

I've put together a [small example on github](https://github.com/tonyjunkes/fw1-lifecycle-example) that ties along with my examples here. It also includes a small example of using the bean factory and lifecycle methods.

Cheers.
