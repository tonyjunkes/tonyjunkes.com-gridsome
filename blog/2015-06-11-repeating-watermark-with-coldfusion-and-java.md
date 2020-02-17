---
title: Repeating Watermark with ColdFusion & Java
path: /repeating-watermark-with-coldfusion-and-java
date: 2015-06-11
summary: Repeating Watermark with ColdFusion & Java
tags: ["ColdFusion", "CFML", "Java"]
---

Let's make some watermarks with Java. This one's been in the backlog of my mind for a good while now. It's really a one situation tool, but it was cool as hell to dig into and figure out how to do it.

I had to apply a watermark on a pretty large sum of images. ColdFusion makes this fairly easy. In the given scenario though, it wasn't just a watermark; but a repeating watermark on a 45 degree. Still fairly simple with CFML I'm sure (?), but I didn't go that route. I called on some Java classes instead. In the past, I've had questionable experiences with ColdFusion's image functions when doing heavy lifting on a large amount of images that might need to be resized and altered in more than one way. To be honest, I'm not sure how today's engine performs. I use Lucee these days and I couldn't speak for how it's functions stand up either. Something to compare in the future I suppose.

#### So here's a component that spins up some magic...

```js
component name="Watermark"
	output="false"
{
	public any function init() {
		variables.AlphaComposite = createObject("java", "java.awt.AlphaComposite");
		variables.Color = createObject("java", "java.awt.Color");
		variables.Font = createObject("java", "java.awt.Font");
		variables.jFile = createObject("java", "java.io.File");
		variables.AffineTransform = createObject("java", "java.awt.geom.AffineTransform");
		variables.ImageIO = createObject("java", "javax.imageio.ImageIO");

		return this;
	}

	public void function addTextWatermark(
		required string text,
		required string srcImgPath,
		required string destImgPath
	) {
		try {
			// Read in the image
			var srcImg = variables.ImageIO.read(variables.jFile.init(arguments.srcImgPath));
			var g2d = srcImg.getGraphics();
			
			// Create our font properties for the supplied text and rotate it 45 degress
			var alphaChannel = variables.AlphaComposite.getInstance(variables.AlphaComposite.SRC_OVER, 0.3);
			g2d.setComposite(alphaChannel);
			g2d.setColor(variables.Color.WHITE);
			var aft = variables.AffineTransform;
			aft.rotate(45 * pi() / 180);
			var font = variables.Font.init("Arial", variables.Font.BOLD, 64);
			g2d.setFont(font.deriveFont(aft));
			
			// Get font dimensions
			var fontMetrics = g2d.getFontMetrics();
			var rect = fontMetrics.getStringBounds(arguments.text, g2d);
			
			// Get the x and y factors
			var xFactor = cos(45 * pi() / 180);
			var yFactor = sin(45 * pi() / 180);

			// Crunch through positions to place our watermark text
			for (var x = 0; x < srcImg.getWidth(); x += rect.getWidth() * xFactor + rect.getHeight() * yFactor) {
				for (var y = -60 * yFactor; y < srcImg.getHeight(); y += rect.getWidth() * yFactor + rect.getHeight() * xFactor) {
					g2d.drawString(arguments.text, x, y);
				}
			}

		    // Write file and clean up
		    variables.ImageIO.write(srcImg, "png", variables.jFile.init(arguments.destImgPath));
		    g2d.dispose();
		}
		catch(any e) {
			throw(e.message);
		}
	}
}
```

In a nutshell, it takes a absolute path to a given image, the text to be layered over said image and a destination path (and name) for the result image, processes and creates a newly watermarked image.

So let's break this bad boy down.

```java
<cfscript>
	variables.AlphaComposite = createObject("java", "java.awt.AlphaComposite");
	variables.Color = createObject("java", "java.awt.Color");
	variables.Font = createObject("java", "java.awt.Font");
	variables.jFile = createObject("java", "java.io.File");
	variables.AffineTransform = createObject("java", "java.awt.geom.AffineTransform");
	variables.ImageIO = createObject("java", "javax.imageio.ImageIO");
</cfscript>
```

For separation, I put all of my Java class calls in the init() function to be passed through the variables scope. File and ImageIO are used to create the Graphics2D class object that will be used to create and apply the watermark to. AlphaComposite, Color and Font are used with the result object from ImageIO to actually make the watermark. AffineTransform will help get the 45 degree turn to the watermark text.

```java
<cfscript>
	// Create our font properties for the supplied text and rotate it 45 degress
	var alphaChannel = variables.AlphaComposite.getInstance(variables.AlphaComposite.SRC_OVER, 0.3);
	g2d.setComposite(alphaChannel);
	g2d.setColor(variables.Color.WHITE);
	var aft = variables.AffineTransform;
	aft.rotate(45 * pi() / 180);
	var font = variables.Font.init("Arial", variables.Font.BOLD, 64);
	g2d.setFont(font.deriveFont(aft));
</cfscript>
```

Once I have the image object to work with, I create a alpha channel with a low opacity so it gets that faded appearance on the image. Set it as the composite and make the color white. With the AffineTransform object, pass in a radian value of the degrees I want to turn the text into the rotate() function. Finallym creating the font object and applying the rotation to the object.

```java
<cfscript>
	// Get font dimensions
	var fontMetrics = g2d.getFontMetrics();
	var rect = fontMetrics.getStringBounds(arguments.text, g2d);
	
	// Get the x and y factors
	var xFactor = cos(45 * pi() / 180);
	var yFactor = sin(45 * pi() / 180);
	// Crunch through positions to place our watermark text
	for (var x = 0; x < srcImg.getWidth(); x += rect.getWidth() * xFactor + rect.getHeight() * yFactor) {
		for (var y = -60 * yFactor; y < srcImg.getHeight(); y += rect.getWidth() * yFactor + rect.getHeight() * xFactor) {
		  g2d.drawString(arguments.text, x, y);
		}
	}
</cfscript>
```

I get the dimensions of the text and set the X and Y factors from our angle, then loop over the various coordinates generated from those and the dimensions of the actual image; drawing the text on each pass. That loop was honestly the hardest part for me. I'm not a trig guy by any means. Thankfully there is Google.

The last bit writes the image to the supplied location and clears out our watermark object.

In the end you get something like this...

![Watermark Example](./images/mad.jpg)

Cool stuff. Here's some references I used to put it all together.

- [http://mrbool.com/how-to-create-watermarks-with-java/29773](http://mrbool.com/how-to-create-watermarks-with-java/29773)
- [http://stackoverflow.com/a/22745959/985709](http://stackoverflow.com/a/22745959/985709)
- [http://stackoverflow.com/a/24102883/985709](http://stackoverflow.com/a/24102883/985709)

Here's a [link to my example project on GitHub](https://github.com/tonyjunkes/repeated-watermark-example).

Happy coding.