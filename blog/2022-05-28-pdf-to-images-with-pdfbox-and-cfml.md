---
title: PDF to Images With PDFBox and CFML
path: /pdf-to-images-with-pdfbox-and-cfml
date: 2022-05-28
summary: Breakdown of how to use PDFBox to convert PDF pages to images with CFML.
tags: ["CFML","PDFBox","Java"]
---

Recently, I worked on part of a feature for viewing a PDF, page by page, in the browser. To accomplish this, the PDF is converted into JPG images and displayed in a carousel. In this article, I want to touch on the image conversion and how it can be done using [Apache PDFBox](https://pdfbox.apache.org/).

I've touched on converting PDFs to images in a [past article](https://tonyjunkes.com/blog/formatting-a-color-pdf-to-grayscale/) which also used PDFBox. Not much has changed since then, but it's amazing what you can learn when using it for real-world solutions. Especially when the consideration of performance rears its head, which I'll go over in more detail.

#### Demo Code?

If you'd like to dig right into some runnable code, I've put together a [demo application on GitHub](https://github.com/tonyjunkes/cfml-pdf-to-image-example). The example code is near exact to what I'll go over here, so feel free to follow along with it too.

## CommandBox Setup
---

This demo will set everything up to use [CommandBox](https://www.ortussolutions.com/products/commandbox) for the JAR dependencies and running the server instance.

There are a few dependencies for CommandBox to pull in via `box.json`. They will live in the `/lib` folder.

- PDFBox 2.0.26 (the star of the show)
- FontBox 2.0.26 (a dependency of PDFBox)

```json
// box.json
{
    "name":"CFML PDF to Image Example",
    "dependencies":{
        "pdfbox-2.0.26":"jar:https://search.maven.org/remotecontent?filepath=org/apache/pdfbox/pdfbox/2.0.26/pdfbox-2.0.26.jar",
        "fontbox-2.0.26":"jar:https://search.maven.org/remotecontent?filepath=org/apache/pdfbox/fontbox/2.0.26/fontbox-2.0.26.jar"
    },
    "installPaths":{
        "pdfbox-2.0.26":"lib/pdfbox-2.0.26/",
        "fontbox-2.0.26":"lib/fontbox-2.0.26/"
    }
}
```

From within the project directory, running `box install` will pull down the JAR files from [search.maven.org](https://search.maven.org) and place them in the `/lib` folder.

> Note: At the time of this article, PDFBox 3.0 is still in pre-release. There is a breaking change with at least one of the methods used in this demo, so if you come here looking for answers in the future, just know things may not align. As I recall, the changes are quick to alter. Perhaps I'll touch on it more when it goes gold.

## Application.cfc
---

The `Application.cfc` will be straightforward. We set an array of the JAR file paths for `this.javaSettings` using `directoryList()` and create two mappings. One for where we will store the PDF(s) to be converted (/resources) and the other for the PDF conversion CFC (/components).

```js
// Application.cfc
component {
    this.name = hash(getBaseTemplatePath());
    this.applicationTimeout = createTimeSpan(0, 2, 0, 0);
    this.javaSettings.loadPaths = directoryList(expandPath("/lib"), true, "path", "*.jar");
    this.mappings = {
        "/resources": expandPath("/resources"),
        "/components": expandPath("/components")
    };
}
```

## PDF Service Component
---

There's quite a bit going on here so I'll break it down more below.

As a quick run-through, the function reads in a binary PDF file and gets a catalog and iterator of the pages. Each page is then asynchronously converted to a JPG from a newly loaded version of the file during each iteration. To maintain page order during the async process, the page index is used as an identifier for sorting at the end. An array of binary images is the final result.

#### Why process each page asynchronously?

When working on a real-world application of this, it was quickly noticed that image conversions were taking seconds per page in a typical blocking process. By threading each page, the time to complete could, in theory, only take as long as the longest page conversion. In my case, this was much faster. It's important to keep in mind, however, that processing a large PDF with many pages may require some extra steps to ensure performance and memory consumption are kept in a management state. For example, chunking page processing into X number of thread batches at a time before running the next batch. Your mileage will vary based on your environment and resources.

```js
component displayname="PDF Service"
    output=false
{
    public array function pdfToImage(binary pdfFile) {
        var result = [];
        // Collection for async processes
        var futures = [];

        try {
            var ByteArrayOutputStream = createObject("java", "java.io.ByteArrayOutputStream");
            var ImageIO = createObject("java", "javax.imageio.ImageIO");
            var PDFRenderer = createObject("java", "org.apache.pdfbox.rendering.PDFRenderer");
            var PDDocument = createObject("java", "org.apache.pdfbox.pdmodel.PDDocument");

            // Get the page tree iterator from the document
            var pageTreeDocument = PDDocument.load(arguments.pdfFile);
            var pageTree = pageTreeDocument.getDocumentCatalog().getPages();
            var pageIterator = pageTree.iterator();

            // Process pages
            while (pageIterator.hasNext()) {
                var currentPage = pageIterator.next();
                var pageIndex = pageTree.indexOf(currentPage);
                // Note: PDFBox is not thread safe!
                // A new instance of the PDF is created for each thread
                var threadDocument = PDDocument.load(arguments.pdfFile);

                // Collect each future worker
                futures.append(
                    ((document, index) => {
                        return runAsync(() => {
                            try {
                                // Convert page to BufferedImage
                                var renderer = PDFRenderer.init(document);
                                var pageImage = renderer.renderImage(index);
                                // Write as JPG to OutputStream
                                var imageOS = ByteArrayOutputStream.init();
                                ImageIO.write(pageImage, "jpg", imageOS);
                                // Add results to collection to be sorted later
                                return { index: index, image: imageOS.toByteArray() };
                            }
                            catch(any e) {
                                rethrow;
                            }
                            finally {
                                // Close streams
                                document?.close();
                                imageOS?.close();
                            }
                        });
                    })(threadDocument, pageIndex)
                );
            }

            // Block (await) any further processing until each future has completed
            // Sort pages back to original order and return the images
            result = futures
                .map((future) => arguments.future.get())
                .sort((page1, page2) => arguments.page1.index - arguments.page2.index)
                .map((page) => arguments.page.image);
        }
        catch(any e) {
            // For debugging the example, don't use in production!
            writeDump(e);
        }
        finally {
            // Close streams
            pageTreeDocument?.close();
            threadDocument?.close();
        }

        return result;
    }
}
```

### Breaking Things Down

After the Java classes are defined, the PDF is loaded as a PDFBox `PDDocument` object so we can get a catalog of the pages and an iterator to use for looping over those pages.

```js
var pageTreeDocument = PDDocument.load(arguments.pdfFile);
var pageTree = pageTreeDocument.getDocumentCatalog().getPages();
var pageIterator = pageTree.iterator();
```

The iterator gives us a handy way to check for any pages on the next iteration, and a method to call that page. Perfect for a while loop. The page tree catalog can then be used to get the index of the current page. This is necessary not just for sorting later but also for referencing the page inside of a thread.

```js
while (pageIterator.hasNext()) {
    var currentPage = pageIterator.next();
    var pageIndex = pageTree.indexOf(currentPage);
    //... process logic here ...
}
```

Now for an important step. Because PDFBox is not threadsafe, to process these pages asynchronously, we have to load a new instance of the PDF on each iteration. Skipping this will result in pages rendering in a corrupt state, or not at all.

```js
var threadDocument = PDDocument.load(arguments.pdfFile);
```

Next in the iteration, we can get fancy and append an array of [immediately invoked function expressions](https://helpx.adobe.com/si/coldfusion/developing-applications/building-blocks-of-coldfusion-applications/writing-and-calling-user-defined-functions/creating-user-defined-functions.html#iife) (IIFE) that hold the async result of the image processing.

> Note: IIFEs are supported in Lucee 5 and ColdFusion 2021. To accomplish this in ColdFusion 2018, set the function expression to a variable ahead of time and call it on each iteration like any normal function.

The function takes the document instance created on each iteration and the index of the page that's next to be processed as arguments. Then returns a `runAsync` call that creates a `PDFRenderer` object of the document, renders the specific page, and writes it as a JPG to a `ByteArrayOutputStream`. Each `runASync` call returns a struct referencing the page index and the output stream as a byte array.

```js
futures.append(
    ((document, index) => {
        return runAsync(() => {
            try {
                // Convert page to BufferedImage
                var renderer = PDFRenderer.init(document);
                var pageImage = renderer.renderImage(index);
                // Write as JPG to OutputStream
                var imageOS = ByteArrayOutputStream.init();
                ImageIO.write(pageImage, "jpg", imageOS);
                // Add results to collection to be sorted later
                return { index: index, image: imageOS.toByteArray() };
            }
            catch(any e) {
                rethrow;
            }
            finally {
                // Close streams
                document?.close();
                imageOS?.close();
            }
        });
    })(threadDocument, pageIndex)
);
```

Once each page has been passed to an async process, the collection then waits for them to finish and set the resultant array of structs via `arrayMap`. The result is then sorted by page index, calling `arrayMap` again to map the final result to an array of PDF page JPGs as byte arrays.

```js
result = futures
    .map((future) => arguments.future.get())
    .sort((page1, page2) => arguments.page1.index - arguments.page2.index)
    .map((page) => arguments.page.image);
```

One last thing to point out, both in each async call and at the end of the entire process, is to ensure we close both the document instances and the output streams. Forgetting to do so can result in unwanted results and memory leaks. Since these calls are done in the `finally` block of each `try/catch`, safe navigation `?.` is used in case an error occurred before the instance is defined. This makes for smoother object cleanup regardless of success or failure.

```js
document?.close();
imageOS?.close();

pageTreeDocument?.close();
threadDocument?.close();
```

All that's left is to call the function and pass a PDF to it.

```html
<h1>CFML PDF to Image Example</h1>

<!--- File conversion --->
<cfscript>
    PDFService = new components.PDFService();
    pdfFile = fileReadBinary(expandPath("/resources/sample.pdf"));
    pages = PDFService.pdfToImage(pdfFile = pdfFile);
</cfscript>

<!--- Draw results to screen --->
<cfloop array="#pages#" item="page" index="index">
    <cfoutput>
        <h3>Page #index#</h3>
        <img src="data:image/jpg;base64,#binaryEncode(page, "base64")#" alt="Page #index#" />
    </cfoutput>
</cfloop>
```

## Done!

That's all there is to it. PDFBox is such a powerful tool when working with and manipulating PDFs. This kind of tooling doesn't even scratch the surface of what it's capable of. I hope this was a useful exercise, and at the very least, provides an example of how leveraging Java can make your CFML applications that much more efficient.

Cheers & Happy Coding!