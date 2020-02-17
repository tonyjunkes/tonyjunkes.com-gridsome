---
title: Parsing XML With jsoup In CFML - A Simple Example
path: /parsing-xml-with-jsoup-in-cfml
date: 2017-09-23
summary: Parsing XML With jsoup In CFML
tags: ["CFML","jsoup"]
---

At work, we've had some tasks lately to build out database schemas for populating rates tied to a given item and category. The rates themselves are in a PDF file. Much of the work has involved some form of manual entry or small copy/paste conversions into the build script. As it stands, these manual tasks can take anywhere from a day, to a day and a half, to complete. Not at all ideal. Especially with about ~40 more builds to go.

So I started an initial investigation into building a script. Something that would at least cut our build generation time drastically. The main piece was extracting rates and other useful info from a given PDF. I've never done this before in CFML but, I was delighted to see that the data extracted was XML. Well, not delighted it was XML but more that it was workable data!

#### Extracting PDF Text

ColdFusion makes this easy with the `<cfpdf>` tag. One line of code gives me XML that I can work with.

```js
cfpdf(name="rawXML" action="extracttext", source="./path/to/my.pdf");
```

#### Parsing With jsoup

I'll admit, this part had me stumped for a minute (hence this article!). I knew I could use jsoup to accomplish the task however, I was getting strange results back.

Consider this XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<DocText>
    <Pages>
        <Page number="1">Some text here.</Page>
        <Page number="2">Some more text here.</Page>
        <Page number="3">Some other text here.</Page>
    </Pages>
</DocText>
```

Visually, it appears simple enough to extract the data inside of each `<Page>` with a selector.

```js
// jsoup object
Jsoup = createObject("java", "org.jsoup.Jsoup");
// Extract XML data
cfpdf(name="rawXML" action="extracttext", source="./path/to/my.pdf");
// Parse the data into a jsoup Document
doc = Jsoup.parse(rawXML);
// Select the text in each <Page> element
pages = doc.select("Pages"); // Returns an array Elements
```

But this wasn't so.

When I access a node in the array returned, I would see a string like this:

```html
Some text here.<Page number="2">Some more text here. Some other text here.</Page></Page></Page>
```

This is no good when I need each node separate for further extraction of data.

I may have been doing something wrong with the above attempt, but it turns out jsoup has an XML parser that can be passed to the `parse()` method. All that's needed to make this available is to create an instance of jsoup's `Parser` class.

Here's a final example that also includes iterating over the array of Element objects containing the string values I need.

```js
Jsoup = createObject("java", "org.jsoup.Jsoup");
Parser = createObject("java", "org.jsoup.parser.Parser");

cfpdf(name="rawXML" action="extracttext", source="./path/to/my.pdf");
doc = Jsoup.parse(rawXML, "", Parser.xmlParser());
doc.select("Pages").each(function(page) {
    writeDump(page.text());
});
```

So lesson here is to use jsoup's xmlParser() when parsing XML. Regular HTML works just fine without declaring the parser. Hope this helps.

Cheers.
