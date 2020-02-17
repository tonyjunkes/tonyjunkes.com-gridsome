---
title: Using arrayEach() to Generate JSON for CKEditor's Image Browser Plugin
path: /using-arrayeach-to-generate-json-for-ckeditors-image-browser-plugin
date: 2013-11-22
summary: Using arrayEach() to Generate JSON for CKEditor's Image Browser Plugin
tags: ["ColdFusion", "CFML", "CKEditor"]
---

A few months ago I had a client that requested basic abilities to manipulate content on their website I was building with ColdFusion and FW/1. They really didn't require anything crazy and it was an afterthought requested when I was more or less done. To ease into a CMS-like setup from the original static structure, I threw all the content into a database table, whipped up some services to call said content and chose to use [CKEditor](http://ckeditor.com/) to manipulate that data from a form. One of the major bits the client required was using images. While uploading and manipulating images on the server-side is simple enough, I hadn't really used CKEditor for handling images on the server.

By default, CKEditor's option to apply an image requires you to supply a URL location to a given image. This didn't seem ideal considering the client was going to be using their own images and I didn't want them to have to remember any directory structures and paths to form a worthy URL to their images on the server.

Looking through the plugins list on the CKEditor site I came across the [Image Browser Addon](http://ckeditor.com/addon/imagebrowser) which allows you to view and select images from a selected directory on the server. It also allows for a hierarchy of folders containing images so you can have various "categories" to choose from if need be. In order to populate this list, you need to pass a URL to a JSON string (file or remote) containing paths, image names and other folders (if any). Since I needed something dynamic, this was a perfect time for a remote CFC to populate the JSON to reflect a accurate list of images on the sever.

There's most likely more than one way to skin this cat but in the beginning I wasn't getting the desired output of structure. I then decided to give ColdFusion 10's arrayEach() a shot and it worked like a charm!

## Code!

Firstly, the expected JSON would look something like this...

```json
[
        {
                "image": "/image1_200x150.jpg",
                "thumb": "/image1_thumb.jpg",
                "folder": "Small"
        },
        {
                "image": "/image2_200x150.jpg",
                "thumb": "/image2_thumb.jpg",
                "folder": "Large"
        }
]
```

It consists of an array that contains key-value pairs of the image name / path, a thumb version of the previous and a folder that it is in. For my example I am not using thumb. All you really need to get things running is "image".

Now here's the remote CFC for creating the JSON...

```js
component name="ImageBrowser"
	output="false"
{
	remote array function list(required string dir = "")
		output="false"
		returnformat="JSON"
	{
		var length = 0;
		var path = var image = var folder = "";
		var result = [];
 
		arrayEach(
			directoryList(expandPath("./#ARGUMENTS.dir#"), true, "path"),
			function(x) {
				length = listLen(x, "\/");
				if (reFind("\w.*\.(jpg|png|gif|bmp)", listGetAt(x, length, "\/"))) {
					image = listGetAt(x, length, "\/");
					path = reMatch("(#listGetAt(dir, 1, "\/")#).*", reReplace(listDeleteAt(x, length, "\/"), "\\", "/", "ALL"))[1];
					folder = listGetAt(x, length - 1, "\/");
					arrayAppend(result, {"image": "/#path#/#image#", "folder": "#folder#"});
				}
			}
		);
 
		return result;
	}
}
```

Let's see if I can break this down ;) I'll skip any major self-explanatory parts. Above I have a remote function that will return JSON. It takes one parameter which is the directory path beginning from the root to the folder you wish to search through. For example - "/assets/img/content".

Then I use arrayEach() and pass in a array of paths, to the images, created with directoryList() and expandPath(); to locate the dir location. Notice in directoryList() I specify "true" as a parameter value. That's saying yes to check through sub-folders. This would be required to get images in any other folder. I then also specify "path" which gives me an array of string type paths to all of the files found in the list. This value will be used to strip out the different values required in the JSON data.

Now that arrayEach() is cycling through the array, I use reFind() to make sure the only string data I manipulate ends with a image type file extension. Because the array values are paths, I use list functions to pull out my image, path and folder values with a delimiter of "/" and set them to variables. Notice in the "path" variable I reference the "dir" variable. This is the same "ARGUMENTS.dir" value but because arrayEach() uses a anonymous function, it has it's own scopes. Therefore we have to have ColdFusion do some climbing through the chain for us to locate the value we need.

Finally I finish with appending the values wrapped in a struct to an array. Notice the keys are in quotes. This is to preserve their case when outputting as JSON. Otherwise it will output in caps and be invalid JSON (Thanks Adobe :/). At this point the returned data will be nicely formed JSON ready for use.

To test that our data is correct you can use your browser with a URL similar to - `http://mysite.com/path/to/ImageBrowser.cfc?method=list&dir=/assets/images`

To set up CKEditor with our text area and where the addon looks for our JSON, you might do something like this...

```js
CKEDITOR.replace('textareaId', {
    "extraPlugins": "imagebrowser",
    "imageBrowser_listUrl": "/ImageBrowser.cfc?method=list&dir=/assets/images"
});
```

When you use CKEditor now, you can click on the image icon and select the "Browse Server" option which will list the images and folders to search through from the JSON we created.
There we go. At the time, it was a fun little challenge.

You can find a [example demo over on GitHub](https://github.com/tonyjunkes/cf-ckeditor-imagebrowser)

Cheers.