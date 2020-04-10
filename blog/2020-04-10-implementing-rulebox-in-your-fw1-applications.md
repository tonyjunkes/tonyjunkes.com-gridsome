---
title: Implementing RuleBox In Your FW/1 Applications
path: /implementing-rulebox-in-your-fw1-applications
date: 2020-04-10
summary: A rundown on how to implement RuleBox in your FW/1 applications as a Subsystem.
tags: ["CFML","FW1","RuleBox"]
---

This one has been a long time coming. While attending the Adobe ColdFusion Summit 2019, I was able to catch [Luis Majano's talk](https://drive.google.com/file/d/1imx6dQw5fib748gomGcxMDBOQkl7Jl7x/view) on [RuleBox](https://github.com/coldbox-modules/rulebox). From the moment it's features started to click in my mind, I thought this would be cool to use in Framework One.

RuleBox is developed to integrate with ColdBox applications as a module but, similar to some of my other articles and pet projects, I am a fan of taking *Box modules/libs and wiring them up to play nice with FW/1. Sometimes it works well, other times it doesn't. In the case of RuleBox, I was delighted to see that there was minimal effort to get working as a Subsystem.

Per one of the session slides, RuleBox is:
> - A modern and natural language rule engine
> - Inspired by RuleBook (Java Project)
> - Rules are written in the RuleBox Domain Specific Language (DSL)
>   - Modeled after Given-When-Then
>   - Dynamic and Expressive
>   - Testable and Mockable

It's not my intention to go into much detail of the hows and whys so be sure to check out the [session slides](https://drive.google.com/file/d/1imx6dQw5fib748gomGcxMDBOQkl7Jl7x/view) and the [project](https://github.com/coldbox-modules/rulebox) itself for more info.

Let's get into some code.

> Note: Going forward, it will be assumed that you know how to scaffold out the basic innards of an [FW/1](http://framework-one.github.io/) app (controllers, views, etc.) with a little help from [CommandBox](https://www.ortussolutions.com/products/commandbox). We'll be focusing on the key blocks needed to wire everything together. To see a full functioning example, check out my [FW/1 &amp RuleBox example app on GitHub](https://github.com/tonyjunkes/fw1-rulebox-example).

## CommandBox Dependencies
---

For all of this to come together, we need a few dependencies for CommandBox to pull in:

- FW/1
- RuleBox
- LogBox (For RuleBox)

```json
// box.json
{
    "name":"fw1-rulebox-example",
    "dependencies":{
        "fw1":"be",
        "logbox":"^5.6.2",
        "rulebox":"^1.0.0"
    },
    "installPaths":{
        "fw1":"framework/",
        "logbox":"subsystems/logbox/",
        "rulebox":"subsystems/rulebox/"
    }
}
```

From there, your project is a `box install` away from having the core requirements to get started.

> Note: For bonus points, we could also include TestBox as a dev dependency so that we can integrate the existing test cases (and write our own!) with our FW/1 application. I won't cover that in this article but I have included tests in the example app mentioned earlier.

## FW/1 Framework Settings (Application.cfc)
---

Now let's cover the requirements in `Application.cfc`.

## Mappings

To correctly resolve the module locations for DI/1, we need to add mappings pointing to `LogBox` and `RuleBox` in the `/subsystems` directory.

```js
// Application.cfc
this.mappings = {
    "/logbox" = expandPath( "./subsystems/logbox" ),
    "/rulebox" = expandPath( "./subsystems/rulebox" )
};
```

## Framework Settings (`variables.framework = {}`)

Before going into the parent bean factory settings in `variables.framework`, let's cover setting up the subsystem bean factories for the *Box modules. These will live in the `subsystems: {}` struct in `variables.framework`.

```js
variables.framework = {
    // Some framework settings
    subsystems: {
        // Our module settings
    }
}
```

## LogBox

If you're unfamiliar with using LogBox in FW/1, you can check out another article of mine for a more detailed setup: [Using LogBox For Logging In FW/1](https://tonyjunkes.com/using-logbox-for-logging-in-fw1/)

First, we need to define a configuration file for `LogBox` to use. This can go anywhere you desire. In this case, we will reference `LogBoxConfig.cfc` in a `/conf` directory located at the root of the application. This will configure `LogBox` to write logs to a file called `fw1rulebox.log` in a `/logs` directory.

```js
component {
    void function configure() {
        logBox = {
            appenders = {
                fw1rulebox = {
                    class = "logbox.system.logging.appenders.RollingFileAppender",
                    properties = {
                        filePath = "./logs",
                        autoExpand = true,
                        fileMaxSize = 3000,
                        fileMaxArchives = 5
                    }
                }
            },
            root = { appenders = "*" }
        };
    }
}
```

Next in `variables.framework.subsystems`, we define a `logbox: {}` struct where we set the `diConfig` settings in a load listener.

```js
logbox: {
    diConfig: {
        loadListener: ( di1 ) => {
            di1.declare( "LogBoxConfig" ).instanceOf( "logbox.system.logging.config.LogBoxConfig" )
                .withOverrides( { CFCConfigPath: "conf.LogBoxConfig" } )
                .done()
                .declare( "LogBox" ).instanceOf( "logbox.system.logging.LogBox" )
                .withOverrides( { config: di1.getBean( "LogBoxConfig" ) } );
        }
    }
}
```

Two things are happening here:
- Declare a bean called `LogBoxConfig` that points to an instance of the LogBoxConfig CFC and loads in the configuration file defined in `/conf` as a dependency.
- Declare a bean called `LogBox` that points to an instance of the LogBox CFC and loads in the `LogBoxConfig` bean as a dependency.

## RuleBox

Now in the same `subsystems: {}` struct, we will add in our `RuleBox` settings.

```js
rulebox: {
    diLocations: [ "/models" ],
    diConfig: {
        transientPattern: "^(Rule|Result)",
        singulars: { models: "@rulebox" },
        loadListener: ( di1 ) => {
            di1.declare( "Builder@rulebox" ).asValue({
                ruleBook: ( name ) => {
                    return di1.getBean( "RuleBook", { name: name } );
                },
                rule: ( name ) => {
                    return di1.getBean( "Rule", { name: name } );
                }
            });
        }
    }
}
```

Let's break down what's going on above:

### Location

`diLocations: [ "/models" ]` lets DI/1 know to scan the `/models` directory within the RuleBox module and add all of the CFCs discovered, to be added to the subsystem bean factory. There's more to this, however, as we do not want to just blindly load up CFCs and assume what type of object they should be treated as. This is important and things simply will not work as expected come result time if we do not identify our transient/singleton objects appropriately.

### Defined Convention

That's where `transientPattern: "^(Rule|Result)"` comes into play. If you open up the CFCs in the RuleBox project, you'll notice one of two things. The CFC will have a `singleton` attribute declared or it will not.

> To the best of my knowledge, this decides whether an object is created in WireBox as a transient or, the more obvious, a singleton when being used in ColdBox. The default policy in WireBox seems to be to make any CFC a transient unless defined/mapped otherwise. In FW/1 it's the opposite and any CFCs that do not live in a /beans directory will be assumed as a singleton object.

So,

- `transientPattern: "^(Rule|Result)"` tells DI/1 to match any CFCs that contain "Rule" or "Result" in their name and define them as transient objects.
- Everything else outside of FW/1's assumed conventions will be defined as singletons.

`singulars: { models: "@rulebox" }` applies what is essentially a mass alias on every object in the bean factory. For example, a `Rule` object can be referenced from the bean factory as `Rule@RuleBox`. This will be necessary for a later step as the module CFCs expect a WireBox object for calling other objects by these naming conventions. More on that later.

### Overriding

The last piece is defined in a load listener.

```js
loadListener: ( di1 ) => {
    di1.declare( "Builder@rulebox" ).asValue({
        ruleBook: ( name ) => {
            return di1.getBean( "RuleBook", { name: name } );
        },
        rule: ( name ) => {
            return di1.getBean( "Rule", { name: name } );
        }
    });
}
```

The `Builder` CFC in the module does not have `accessors="true"` but expects a WireBox object as a dependency. Unfortunately, without this declaration, DI/1 will not know to inject anything. To get around this, we declare an object called `Builder@rulebox` and set it to a struct containing keys that mimic its the `Builder` CFC methods as function expressions; returning the same expected results as the original CFC.

## Parent Bean Factory

There's one last block of configuration needed to tie everything together. This will all be defined in the parent bean factory config of `variables.framework`.

```js
diLocations: [ "/model" ],
diConfig: {
    transients: [ "rules" ],
    loadListener: ( di1 ) => {
        // Create an impersonation of WireBox :D
        di1.declare( "WireBox" ).asValue({
            getInstance: ( name, initArguments ) => {
                // Parse object@module to get subsystem
                var module = name.listToArray( "@" ).last();
                return getBeanFactory( module ).getBean( name, initArguments );
            }
        });
        // Pull in the root logger as a core citizen
        di1.declare( "Logger" ).asValue( getBeanFactory( "logbox" ).getBean( "LogBox" ).getRootLogger() );
    }
}
```

### More Conventions

As a convention that plays nice with FW/1, we will be using a `/model` directory as a home for our `Rule` beans. `diLocations: [ "/model" ]` will tell DI/1 to wire up anything we place there.

Any rules that we create for RuleBox to use will need to be treated as transients so we will define `transients: [ "rules" ]`. Based on this convention, we will be storing our `Rule` CFCs in a `/rules` directory within `/model`.

### Fake It To Make It

Within our load listener, we need to define two objects and make them available from the parent bean factory.

First, we need to create a WireBox object to mimic what would be done for RuleBox in a ColdBox application. As mentioned earlier, the module CFCs expect it as a dependency for pulling in sibling objects. Instead of including WireBox in our app and going through the extra steps to set it up alongside DI/1, we will declare a WireBox object _in_ DI/1 that stores a function expression mimicking the `getInstance()` method. This method will, in turn, call the RuleBox subsystem bean factory to retrieve any needed objects.

```js
di1.declare( "WireBox" ).asValue({
    getInstance: ( name, initArguments ) => {
        var module = name.listToArray( "@" ).last();
        return getBeanFactory( module ).getBean( name, initArguments );
    }
});
```

Lastly, we declare a "Logger" bean that stores the root logger from the LogBox subsystem bean factory.


```js
di1.declare( "Logger" ).asValue( getBeanFactory( "logbox" ).getBean( "LogBox" ).getRootLogger() );
```

## Putting It All Together
---

Now that we've discussed wiring everything up, let's put it to work! As a comparable example, I am going to use an example found in the [RuleBox README](https://github.com/coldbox-modules/rulebox#a-more-complex-scenario) where home loans are calculated using specific rules.

## The Model

Our model is simple, we will be working with 2 forms of transient objects:

- Applicants
- Rules (RuleBooks)

### Applicant

First, we will create our Applicant bean that will contain properties that represent data to run against our rules.

```js
// Applicant Bean: /model/beans/Applicant.cfc

component accessors="true" {
    property creditScore;
    property cashOnHand;
    property firstTimeHomeBuyer;

    function init( creditScore, cashOnHand, firstTimeHomeBuyer ) {
        variables.creditScore = arguments.creditScore;
        variables.cashOnHand = arguments.cashOnHand;
        variables.firstTimeHomeBuyer = arguments.firstTimeHomeBuyer;
        return this;
    }
}
```

### RuleBook

Next, we lay out our RuleBook rules. This extends RuleBox's RuleBook.cfc in order to run an Applicant's properties against the defined rules.

```js
// Home Loan Rate Rule Book: /model/rules/HomeLoanRateRuleBook.cfc

component extends="rulebox.models.RuleBook" {
    function defineRules() {
        // credit score under 600 gets a 4x rate increase
        addRule(
            newRule( "credit score under 600 gets 4x rate increase" )
            .when( ( facts ) => { return facts.applicant.getCreditScore() < 600; } )
            .then( ( facts, result ) => { result.setValue( result.getValue() * 4 ); } )
            .stop()
        );

        // credit score between 600 and 700 pays a 1 point increase
        addRule(
            newRule( "between 600 and 700 pays 1 point" )
            .when( ( facts ) => { return facts.applicant.getCreditScore() < 700; } )
            .then( ( facts, result ) => { result.setValue( result.getValue() + 1 ); } )
        );

        // credit score is 700 and they have at least $25,000 cash on hand
        addRule(
            newRule( "credit score is 700 and they have at least $25,000 cash on hand" )
            .when(( facts ) => {
                return ( facts.applicant.getCreditScore() >= 700 && facts.applicant.getCashOnHand() >= 25000 );
            })
            .then( ( facts, result ) => { result.setValue( result.getValue() - 0.25 ); } )
        );

        // first time homebuyers get 20% off their rate (except if they have a creditScore < 600)
        addRule(
            newRule( "first time homebuyers get 20% off their rate (except if they have a creditScore < 600)" )
            .when( ( facts ) => { return facts.applicant.getFirstTimeHomeBuyer(); } )
            .then( ( facts, result ) => { result.setValue( result.getValue() * 0.80 ); } )
        );
    }
}
```

## Handling Rules & Processing

With the model defined, we can put together a controller to take an Applicant, run it against a RuleBook and return the results to the user.

### Controller

For the handler, we will create a controller with a default item to handle asking the bean factory for an `Applicant`, populate it with data and pass it to our `HomeLoanRateRuleBook` RuleBook which we have added as a dependency via property injection. We then run and set the results to a request context variable to return to the user.

```js
component accessors="true"
    output="false"
{
    property BeanFactory;
    property HomeLoanRateRuleBook;

    void function default( struct rc = {} ) {
        var ruleBook = variables.HomeLoanRateRuleBook;
        var applicant = variables.BeanFactory.getBean(
            "Applicant", { creditScore: 650, cashOnHand: 20000, firstTimeHomeBuyer: true }
        );
        rc.homeLoans = ruleBook
            .withDefaultResult( 4.5 )
            .given( "applicant", applicant )
            .run();
    }
}
```

### View

Our view can be simple as we're just looking to see some basic results in this example. In the controller we defined a default result of "4.5", but if we follow the logic setup in our RuleBook and run our Applicant data against it, we should see a result of "4.4".

```html
<cfdump var="#rc.homeLoans.getResult().getDefaultValue()#" label="Default value should be 4.5">
<cfdump var="#rc.homeLoans.getResult().getValue()#" label="New value should be 4.4">
```

## Done!
---

There we have it. A bit of a long-winded post, but incorporating RuleBox in FW/1 was actually much simpler than expected. As mentioned above, you can find a working [FW/1 &amp RuleBox example application on GitHub](https://github.com/tonyjunkes/fw1-rulebox-example). Complete with tests you can run in CommandBox or from the browser (Note: I've only tested it with Lucee 5).

Thanks to Luis Majano for putting together such a cool project.

Cheers & Happy Coding!