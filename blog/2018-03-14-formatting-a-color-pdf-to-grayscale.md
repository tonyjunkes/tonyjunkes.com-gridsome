---
title: Formatting a Color PDF to Grayscale - An Exercise in Leveraging Java
path: /formatting-a-color-pdf-to-grayscale
date: 2018-03-14
summary: Formatting a Color PDF to Grayscale
tags: ["CFML","PDFBox","Java"]
---

A week or two ago, it was asked in the CFML slack if there was a smooth (easy?) way to convert an RGB formatted PDF to grayscale. I didn't know of any simple method to reach for without using a 3rd party tool or paid solution (or both!). I don't know much about ColdFusion or Lucee's implementations, let alone if they can do it; but my understanding is they only really handle RGB color. It caught my interest, so I figured I'd see what was capable. I took a look at 2 Java libraries: the older (free) version of iText and PDFBox. After some playing around with both, it seems PDFBox would do the job without shelling out money for a newer version of iText for the same abilities.

## Some Backstory - What Is PDFBox?

From the project home page:

> The Apache PDFBox&reg; library is an open source Java tool for working with PDF documents. This project allows creation of new PDF documents, manipulation of existing documents and the ability to extract content from documents. Apache PDFBox also includes several command-line utilities. Apache PDFBox is published under the Apache License v2.0. [Ref](https://pdfbox.apache.org/)

Needless to say, you can do a laundry list of things with PDFs.

## Let's See Some Code...

So our goal of converting a color PDF to grayscale will consist of reading in the PDF, iterating through the pages and converting them to grayscale, writing those pages out as images and then reading those images back into a newly created PDF.

### Java Dependencies

Let's lay out some dependencies and have CommandBox do the heavy lifting. A super-cool-awesome feature of CommandBox is being able to define dependencies with a `JAR endpoint` that points to a URL resource path. This allows us to pull Java libs hosted by the de facto sources like Maven Central.

```json
// box.json
{
    "name":"pdf-grayscale-example",
    "version":"0.0.1",
    "description":"An example of how to convert a RGB formatted PDF to grayscale.",
    "dependencies":{
        "pdfbox-2.0.8":"jar:https://search.maven.org/remotecontent?filepath=org/apache/pdfbox/pdfbox/2.0.8/pdfbox-2.0.8.jar",
        "pdfbox-tools-2.0.8":"jar:https://search.maven.org/remotecontent?filepath=org/apache/pdfbox/pdfbox-tools/2.0.8/pdfbox-tools-2.0.8.jar",
        "fontbox-2.0.8":"jar:https://search.maven.org/remotecontent?filepath=org/apache/pdfbox/fontbox/2.0.8/fontbox-2.0.8.jar"
    },
    "installPaths":{
        "pdfbox-2.0.8":"app/lib/pdfbox-2.0.8",
        "pdfbox-tools-2.0.8":"app/lib/pdfbox-tools-2.0.8",
        "fontbox-2.0.8":"app/lib/fontbox-2.0.8"
    }
}
```

There are 3 libs we need for this:

- The main PDFBox library.
- The "tools" extension of it which includes the image classes we need.
- A font extension that is extended by some of the classes we'll be using.

To make use of the libraries, we'll need to include `this.javaSettings` in our `Application.cfc`.

```js
// Application.cfc
this.javaSettings.loadPaths = directoryList( expandPath( "path/to/libs" ) );
```

### The CFC

Now we can define our component that will house a few functions for doing the magic.

First, we will instantiate the required Java classes in the constructor. This will include a handful of PDFBox classes for working with a document, its pages and the image utilities for reading and writing the content.

```js
// CFC Constructor
public PDFColorFormatUtil function init() {
    // PDFBox
    variables.PDDocument = createObject( "java", "org.apache.pdfbox.pdmodel.PDDocument" );
    variables.PDPage = createObject( "java", "org.apache.pdfbox.pdmodel.PDPage" );
    variables.PDPageContentStream = createObject( "java", "org.apache.pdfbox.pdmodel.PDPageContentStream" );
    variables.PDRectangle = createObject( "java", "org.apache.pdfbox.pdmodel.common.PDRectangle" );
    variables.PDImageXObject = createObject( "java", "org.apache.pdfbox.pdmodel.graphics.image.PDImageXObject" );
    variables.PDFRenderer = createObject( "java", "org.apache.pdfbox.rendering.PDFRenderer" );
    variables.ImageType = createObject( "java", "org.apache.pdfbox.rendering.ImageType" );
    // PDFBox Tools
    variables.ImageIOUtil = createObject( "java", "org.apache.pdfbox.tools.imageio.ImageIOUtil" );
    // io
    variables.JFile = createObject( "java", "java.io.File" );

    return this;
}
```

Next, let's look at a function to convert a PDF's pages to images. We're going to read our original (src) PDF into a PDFBox document, pass it to a renderer where we loop through the pages and convert them to buffered images and write them out as grayscale jpg. We also set the DPI for the sake of keeping options open for use.

> Note: Keep in mind, writing out images with a high DPI may result in individual images (pages) that are larger in file size than their original state.

```js
// CFC pdfToImage()
public void function pdfToImage( required string src, required string destination, numeric dpi = 300 ) {
    // Read in the PDF as a PDDocument object
    var document = variables.PDDocument.load( variables.JFile.init( src ) );
    // Pass the Document to a PDFRenderer, get the pages
    var renderer = variables.PDFRenderer.init( document );
    var pdPages = document.getDocumentCatalog().getPages();
    // Get the file name as a naming identifier for the image(s)
    var imageTitle = src.listLast( "\/" ).listFirst( "." );
    // Iterator counter
    var page = 0;
    // Iterate over each page, create a buffered image and write the image out
    for ( var pdPage in pdPages.iterator() ) {
        var bim = renderer.renderImageWithDPI( page, dpi, variables.ImageType.GRAY );
        variables.ImageIOUtil.writeImage( bim, destination & imageTitle & "-" & ++page & ".jpg", dpi );
    }
    document.close();
}
```

Now we need a function to handle reading those images back in as usable content and write out a PDF document.

```js
// CFC imageToPDF()
public void function imageToPDF( required string src, required string destination ) {
    // Define valid PDImageXObject content formats
    var formats = [ "png", "jpg", "jpeg", "gif", "bmp" ];
    // Create blank PDDocument as the new PDF shell
    var document = variables.PDDocument;
    // Read in the directory of images
    var dir = variables.JFile.init( src );
    // Iterate over each image and create a PDF page from it
    for ( var img in dir.listFiles() ) {
        if ( formats.find( img.getName().listLast( "." ) ) ) {
            var imgObj = variables.PDImageXObject.init( document ).createFromFileByContent( img, document );
            var width = imgObj.getWidth();
            var height = imgObj.getHeight();
            var page = variables.PDPage.init( variables.PDRectangle.init( width, height ) );
            document.addPage( page );
            var contentStream = variables.PDPageContentStream.init( document, page );
            contentStream.drawImage( imgObj, 0, 0 );
            contentStream.close();
        }
    }
    document.save( destination );
    document.close();
}
```

Finally, some code to call the component/functions...

```java
<!--- index.cfm --->
<cfscript>
    // Instantiate the CFC
    PDFColorFormatUtil = new path.to.PDFColorFormatUtil();
    // Convert PDF to images
    PDFColorFormatUtil.pdfToImage( "path/to/test.pdf", "path/to/images/", 200 );
    // Convert folder of images into a PDF
    PDFColorFormatUtil.imageToPDF( "path/to/images", "path/to/output/result.pdf" );
</cfscript>
```

That wraps it up. A little bit of effort digging into Java consisting of ~60 lines of code and we have a simple utility for modifying a PDF's color format. There are other options available for the color type; as well as features to do a whole lot more with PDFs. All in all, it was a fun exercise for me.

I've created a small [example application available on GitHub](https://github.com/tonyjunkes/pdf-grayscale-example) for anyone who wants to give it a try.

Happy coding.
