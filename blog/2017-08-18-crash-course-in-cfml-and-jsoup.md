---
title: Crash Course In CFML & jsoup
path: /crash-course-in-cfml-and-jsoup
aliases: ["/blog/crah-course-into-cfml-and-jsoup/", "/blog/crash-course-into-cfml-and-jsoup/"]
date: 2017-08-18
summary: Crash Course In CFML & jsoup
tags: ["CFML","jsoup","Java"]
---

Over the years I've made repeatable use of the jsoup library so I figured it'd be nice to put out a little primer on using it with CFML.

## What Is jsoup?
---

From the official site:

> [jsoup](https://jsoup.org/) is a Java library for working with real-world HTML. It provides a very convenient API for extracting and manipulating data, using the best of DOM, CSS, and jquery-like methods.
>
> jsoup is designed to deal with all varieties of HTML found in the wild; from pristine and validating, to invalid tag-soup; jsoup will create a sensible parse tree.

jsoup allows you to do such things as:

- Scrape and parse HTML from a URL, file, or string
- Find and extract data, using DOM traversal or CSS selectors
- Manipulate the HTML elements, attributes, and text
- Clean user-submitted content against a safe white-list, to prevent XSS attacks
- Output tidy HTML

## Getting Started
---

There are a few ways to go about integrating jsoup into an application.

#### A) Installing With CommandBox

> Note: This assumes CommandBox version 3.7+

From the CommandBox CLI, create a new project directory: `mkdir cfml-jsoup-example`, and `cd` to the new folder. From there run `init cfml-jsoup-example` to create a `box.json` for your project.

```
#> mkdir cfml-jsoup-example
#> cd cfml-jsoup-example
#> init cfml-jsoup-example
``` 

Inside of box.json you want to a dependency with a JAR endpoint that points to the URL where the JAR file is located. The installed JAR will always be homed in a directory named after the JAR file, but you can place that folder in any "root" folder of your choice.

```json
{
    "name":"cfml-jsoup-example",
    "dependencies":{
        "jsoup-1.10.3":"jar:https://jsoup.org/packages/jsoup-1.10.3.jar"
    },
    "installPaths":{
        "jsoup-1.10.3":"lib\\jsoup-1.10.3"
    }
}
```

---

#### B) Installing Manually

To manually install jsoup, you can simply go to the [official site download page](https://jsoup.org/download) and pull down the latest `core library` release. Place the JAR file in a folder of your choice within your project.

---

#### Mapping the JAR In Application.cfc

Now we need to add the JAR to the Java Class Path. You can map it in your project's `Application.cfc` via `this.javaSettings`.

```js
this.javaSettings = { loadPaths: ["./your_dir"] };
```
---
> To learn more on integrating 3rd party Java libraries in CFML, check out the [CFDocs - Java Integration Guide](https://cfdocs.org/java).

## Examples
---

### Parsing Documents

A jsoup document can be a string of HTML-like data or data read in from a file as a string.

#### Parse A Document From An HTML String

```java
<cfscript>
    // Create the jsoup object
    Jsoup = createObject("java", "org.jsoup.Jsoup");

    // HTML string
    html = "<html><head><title>CFML & jsoup Example</title></head><body>Content about CFML and jsoup.</body></html>";
    // Parse the string
    document = Jsoup.parse(html);
    // Extract content
    title = document.title();
    body = document.body().text();

    writeOutput("
        <div>Title: #title#</div>
        <div>Body: #body#</div>
    ");
</cfscript>
```

The example code instantiates the Jsoup class and parses a string of HTML. This returns a [Document class object](https://jsoup.org/apidocs/org/jsoup/nodes/Document.html) that we can act on with its methods.

#### Parsing A Document From HTML Files

Consider the following example HTML...

```html
<!DOCTYPE html>
<html>
    <head>
        <title>CFML & jsoup Example</title>
        <meta charset="UTF-8">
        <meta name="keywords" content="jsoup,cfml,java,html">
        <meta name="description" content="Examples for using CFML and jsoup.">
        <meta name="author" content="@tonyjunkes">
    </head>
    <body>
        <header id="header">Getting Started With CFML & jsoup</header>
        <div>Some content...</div>
        <a href="#">A link to useful info</a>
    </body>
</html>
```

And CFML...

```java
<cfscript>
    // Create the jsoup object
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    // Create the File object
    JFile = createObject("java", "java.io.File");

    // Get the absolute file path
    fileName = expandPath("./path/to/file.html");
    // Parse the File object and extract data
    document = Jsoup.parse(JFile.init(fileName), "utf-8");
    header = document.getElementById("header");

    writeOutput(header.text());
</cfscript>
```

The example code demonstrates using jsoup to parse a Java File object that has the path to the HTML file set as the constructor parameter. This returns a Document to act on with its methods.

#### Parsing An External Source

We can connect to an external source using jsoup's `connect()` method.

```java
<cfscript>
    // Create the jsoup object
    Jsoup = createObject("java", "org.jsoup.Jsoup");

    // Connect
    siteAddress = "https://jsoup.org/";
    document = Jsoup.connect(siteAddress).get();

    // Do things to act on the Document...
    
    // Dump the object
    writeDump(document);
</cfscript>
```

So we take a website URL address and pass it to `Jsoup.connect()` and, so long as the site resolves to a valid page, we are returned a Document object to act on. The example above only dumps the returned object to show various functions available to use on the collected content.

---

### A Deeper Look At Working With Documents

Using the same example HTML file content displayed earlier, we will grab various meta data from a Document object.

#### Reading From the \<head\> Element

```java
<cfscript>
    // Create object, pass in file and parse
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    JFile = createObject("java", "java.io.File");

    fileName = expandPath("./path/to/file.html");
    document = Jsoup.parse(JFile.init(fileName), "utf-8");
    
    title = document.title();
    head = document.head();

    writeOutput("Title: #title#");
    writeDump(head);
</cfscript>
```

Once we have parsed the HTML source, we can access data like `title` or everything in the `<head>` element with `head()`.

#### Getting Metadata With Selectors

From the Document object, we can use the `select()` method and pass in selector syntax, similar to jQuery, as the parameter to match and retrieve the metadata values. 

```java
<cfscript>
    // Create object, pass in file and parse
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    JFile = createObject("java", "java.io.File");

    fileName = expandPath("./path/to/file.html");
    document = Jsoup.parse(JFile.init(fileName), "utf-8");
    // Get metadata
    description = document.select("meta[name=description]").first().attr("content");
    keywords = document.select("meta[name=keywords]").first().attr("content");

    writeOutput("
        <p>Description: #description#<p>
        <p>Keywords: #keywords#</p>
    ");
</cfscript>
```

We pass in a selector parameter, to query meta elements, which returns an [Elements class object](https://jsoup.org/apidocs/org/jsoup/nodes/Element.html). Then we can access it's key (attribute) values using `attr()`. We also have access to various helper methods like `first(), last(), next() & prev()`.

#### Getting the HTML Source From An External Document

We can get the raw HTML source of a Document object by calling a parent method: `html()`.

```java
<cfscript>
    // Create the jsoup object and connect
    Jsoup = createObject("java", "org.jsoup.Jsoup");

    siteAddress = "https://jsoup.org/";
    document = Jsoup.connect(siteAddress).get();

    writeDump(document.html());
</cfscript>
```

The `html()` method is borrowed from the `Elements` class object.

#### Get Link Data

Link attributes and content can be obtained using the same selector methods demonstrated earlier.

```java
<cfscript>
    // Create object, pass in file and parse
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    JFile = createObject("java", "java.io.File");

    fileName = expandPath("./path/to/file.html");
    document = Jsoup.parse(JFile.init(fileName), "utf-8");

    // Get an array of links
    links = document.select("a[href]");
    for (link in links) {
        writeOutput("
            <div>Link: #link.attr("href")#</div>
            <div>Text: #link.text()#</div>
        ");
    }
</cfscript>
```

In this example, we see how to get the `href` value using the `attr()` method selector and also how to obtain the text within the actual `<a>` element by using `text()`.

#### Getting Form Input Data

Once we find the `<form>` element in the document, we can use selectors to iterate and grab `<input>` data.

Consider this HTML...

```html
<!DOCTYPE html>
<html>
    <head>
        <title>CFML & jsoup Example</title>
    </head>
    <body>
        <form id="contact" name="contact" action="/">
            <label>Name:</label>
            <input name="fullname" value="Tony Junkes">
            <label>E-Mail:</label>
            <input name="email" value="fake@email.com">
            <label>Message:</label>
            <textarea name="message">Message here...</textarea>
        </form>
    </body>
</html>
```

And CFML...

```java
<cfscript>
    // Create object, pass in file and parse
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    JFile = createObject("java", "java.io.File");

    fileName = expandPath("./path/to/file.html");
    document = Jsoup.parse(JFile.init(fileName), "utf-8");

    // Get the form and inputs
    contactForm = document.getElementById("contact");  
    inputs = contactForm.getElementsByTag("input");

    // Iterate through the inputs
    for (input in inputs) {
        key = input.attr("name");  
        value = input.attr("value");  
        writeOutput("
            <div>Name: #key#</div>
            <div>Value: #value#</div>
        ");  
    }
</cfscript>
```

So we've used `getElementById()` to find the `<form>` element and then `getElementsByTag()` to grab all of the `<input>` elements within the form. At this point, we can iterate through the array of inputs and use selector methods to act on the data.

---

### Sanitizing HTML Content With jsoup

jsoup provides a collection of classes and methods for sanitizing HTML. Similar to Antisamy, you can use a premade or custom [Whitelist class object](https://jsoup.org/apidocs/org/jsoup/safety/Whitelist.html) that specifies valid and invalid elements in a document. This whitelist object is then passed to a [Cleaner class object](https://jsoup.org/apidocs/org/jsoup/safety/Cleaner.html) which checks the document against the whitelist rules and removes any invalid content.

```java
<cfscript>
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    Whitelist = createObject("java", "org.jsoup.safety.Whitelist");
    Cleaner = createObject("java", "org.jsoup.safety.Cleaner");

    html = "<html><head><title>My title</title></head><body><center>Body content</center></body></html>";
    filter = Whitelist.none();
    valid = Jsoup.isValid(html, filter);

    if (valid) {
        writeOutput("The document is valid!");
    } else {
        invalidData = Jsoup.parse(html);
        writeOutput("The document is not valid!");
        writeDump(invalidData.html());
        cleanDocument = Cleaner.init(filter).clean(invalidData);
        writeOutput("The document has been cleaned.");
        writeDump(cleanDocument.html());
    }
</cfscript>
```

This example takes simple HTML content and passes it to a `Whitelist` that calls the `none()` method. This is a pre-defined Whitelist that restricts any HTML markup inside of the `<body>`. When the populated class is passed to the `Cleaner`, the `clean()` method is called to remove any HTML and leave only valid HTML.

A list of default options includes:

- [none()](https://jsoup.org/apidocs/org/jsoup/safety/Whitelist.html#none--)
- [simpleText()](https://jsoup.org/apidocs/org/jsoup/safety/Whitelist.html#simpleText--)
- [basic()](https://jsoup.org/apidocs/org/jsoup/safety/Whitelist.html#basic--)
- [basicWithImages()](https://jsoup.org/apidocs/org/jsoup/safety/Whitelist.html#basicWithImages--)
- [relaxed()](https://jsoup.org/apidocs/org/jsoup/safety/Whitelist.html#relaxed--)

---

### Extra Credit - A Few In Depth Techniques

Here's a few more examples I thought were worth mentioning because jsoup is so cool.

#### Extract &amp; Replace An HTML Element With It's Content

This example gets the inner content of an `<a>` element and replaces the element with only the content; using a [TextNode class object](https://jsoup.org/apidocs/org/jsoup/nodes/TextNode.html).

```java
<cfscript>
    // Create Java objects
    Jsoup = createObject("java", "org.jsoup.Jsoup");
    TextNode = createObject("java", "org.jsoup.nodes.TextNode");

    // Create some markup...
    html = '<html><head><title>Hello World!</title></head><body><h1>A Header</h1><p>Some content. <a href="##">A cool link.</a></p></body></html>';
    // Parse it into a Jsoup Document
    document = Jsoup.parse(html);

    // Create a Node object
    link = document.select("a").first();
    node = TextNode.init(link.text(), "");
    link.replaceWith(node);

    writeDump(label="Original HTML", var="#html#");
    writeDump(label="Link Text", var="#link.text()#");
    writeDump(label="Modified HTML", var="#document.body().toString()#");
    writeDump(node);
</cfscript>
```

Using a `TextNode`, we can store the content between the `<a>` element. Then call the `replaceWith()` method on the element to switch out the HTML for plain text.

#### Filtering Selected Results With Regex

Using the same `select()` method, we can pass in a regular expression string to filter results by using `~=` instead of `=`.

```java
<cfscript>
    // Create Java objects
    Jsoup = createObject("java", "org.jsoup.Jsoup");

    siteAddress = "https://jsoup.org/";
    document = Jsoup.connect(siteAddress).get();
    links = document.select("a[href~=^((?!##|html).)*$]");

    original = [];
    for (link in document.select("a[href]")) {
        original.append(link.attr("href"));
    }

    filtered = [];
    for (link in links) {
        filtered.append(link.attr("href"));
    }

    // Original links
    writeDump(label="Original Links", var="#original#");
    // Filtered links
    writeDump(label="Filtered Links", var="#filtered#");
</cfscript>
```

In this example, we grab links from the first page of the Jsoup site. The filtered links use a regex to exclude any URLs that contain a `#` or the string `html`.

## Wrapping Up
---

`jsoup` is a super powerful framework for working with and manipulating HTML. The possibilities are endless when working with node structured documents.

For more info on it's classes and methods, check out the [jsoup API Docs](https://jsoup.org/apidocs/).

To help break into these examples, I've put together a little project that can be run from CommandBox. You can find the it at [GitHub - cfml-jsoup-example](https://github.com/tonyjunkes/cfml-jsoup-example).

Cheers!
