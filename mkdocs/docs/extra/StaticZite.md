## Introduction
In this tutorial, we are going to be making a very simple static zite, go through the different options in the `content.json` file, and talk about the ZeroFrame class that is auto-generated when you create an empty zite, what it is used for, and why it is important.

> **Note:** It is recommended to read [The Basics](./?/tutorials/the_basics) tutorial first. It provides useful information on how zites are kept secure, updated, how to keep your zite online, and other important information.

## Creating An Empty Zite
In order to create a zite, first we need to create an empty zite that will generate an address for us. You can very easily do this in the ZeroHello page by going to the menu (denoted by the 3 dots beside the ZeroHello Logo) and clicking `Create new, empty site`. It will create an address for you, along with a directory where you zite's files will be stored and a default `index.html` page. Do not delete this page, because it generates some code for us that will be talked about later in the tutorial. You will be redirected to your new zite by ZeroHello.

## Where Are The Zite's Files Located?
When ZeroHello created your empty zite, it created a new directory for it. This directory is located in the `zeronetroot/data` directory under a folder named by your zite's address. You can find where the ZeroNet root directory is by pressing `show data directory` in ZeroHello's menu.

## The content.json File
The first file we are going to open is the zite's `content.json` file. This file is used to keep your zite secure, and provide options for your zite. [The Basics](./?/tutorials/the_basics) tutorial covers how this file is used to keep your zite secure and from being tampered with. However, this tutorial will cover the options that you can set in this file for your zite. If you are unfamiliar with JSON, you should learn it before going forward with zite development. Trust me, JSON is pretty easy.

### Description and Title
The first thing we are going to do is change the zite's description and title. Look for the description and title fields and change them to what you want. The Title field is what is shown in the title/tab bar and in ZeroHello. The description is used for Search Engines, among other things.

### Ignore Field and Regexes
Next, we are going to modify the `ignore` field. The ignore field is used to tell ZeroNet which files you do not want seeded and sent to peers. Any files/folders that match this ignore field will stay on your computer only, and will not be sent to other peers. This is useful for the git directory, an admin directory, and database files.

Database files (ending in `.db`) are ignored because the database is generated from the json files per client. Clients do not share database files. More information is provided about this in the Database Basics tutorial.

The `ignore` field uses a regular expression to match any folders and files against it. Because of this, it is recommended to learn a little bit about regular expressions. However, here is a simple list of things most used in regular expressions and what they mean:
* `.` - any character
* `*` - zero or more of the character or group that comes before it
* `+` - *one* or more of the character or group that comes before it.
* Putting a letter that isn't a special symbol in regexes means it is a required character.
* `(` and `)` - Creates a group, useful for saying you want zero or more of a word (for example: `(and)*` means zero or more `and` words)
* `|` used in a group to mean *or*. For eample: `(and|or)` means it will match anything that is the word `and` *or* the word `or`.
* `\.` - the dot character. You are escaping the dot. This also works for `\*`, `\|`, `\\` and any other special symbols.

> **NOTE:** It should be noted that a recent update to ZeroNet requires that you must place `.` before `*`, `+`, and `{`. You also cannot have more than 9 of these repetition characters. To fix this, if you are using `.*` for each *or branch* in a group, you can easily put it outside of the group (for example: change `(.*.epub|.*.jpg|.*.jpeg|.*.png)` to `.*(epub|jpg|jpeg|png)`).
> This change was made for security and performance reasons.

Here are some useful patterns used in regular expressions. We will be using some of these in our `ignore` regex.
* To ignore a folder, put `foldername`.
* To ignore a file with a certain extension, put `.*db`
* To ignore files of many extensions, you can use a group like this: `.*(ext1|ext2|ext3)`
* To ignore files, directories, and sub-directories in a folder, use this: `folder/.*/.*`

For our ignore regex, we are going to use this:
```
"ignore": "(data/.*db|data/users/.*/.*|.git|admin)",
```

This will ignore the `.git` folder, the `admin` folder, any database files, and the `data/users` folder and its subdirectories. The reason for the last two will be covered in the Database Basics tutorial.

### Viewport
The final thing we are going to do is add a `viewport` field. This is equivalent to adding the viewport meta tag.

* For mobile-supported zites, set the viewport to `width=device-width, initial-scale=1.0, shrink-to-fit=no`
* For non-mobile-supported zites, you can leave it off.

### content.json Docs
You can find more information about the different options in the `content.json` file from the [ZeroNet Docs](/1Docsfk5uLvuXkL4U13eYDxCm2gzi1fNQq/site_development/content_json/index.html).

## The Index File
We are now done with the `content.json` file. Next, open up the `index.html` file. This is what the index file should look like:

```html
<!DOCTYPE html>

<html>
<head>
	<title>New ZeroNet site!</title>
	<meta charset="utf-8">
	<meta http-equiv="content-type" content="text/html; charset=utf-8" />
	<base href="" target="_top" id="base">
	<script>base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")</script>
</head>
<body>

<div id="out"></div>

<script type="text/javascript" src="js/ZeroFrame.js"></script>
<script>

class Page extends ZeroFrame {
	setSiteInfo(site_info) {
		var out = document.getElementById("out")
		out.innerHTML =
			"Page address: " + site_info.address +
			"<br>- Peers: " + site_info.peers +
			"<br>- Size: " + site_info.settings.size +
			"<br>- Modified: " + (new Date(site_info.content.modified*1000))
	}

	onOpenWebsocket() {
		this.cmd("siteInfo", [], function(site_info) {
			page.setSiteInfo(site_info)
		})
	}

	onRequest(cmd, message) {
		if (cmd == "setSiteInfo")
			this.setSiteInfo(message.params)
		else
			this.log("Unknown incoming message:", cmd)
	}
}
page = new Page()
</script>

</body>
</html>
```

You can see that most of it is just simple html. However, there is the javascript that has the `ZeroFrame` class. This is what we are going to be talking about next.

### The ZeroFrame Class
The ZeroFrame class is used to talk to the ZeroFrame client via an API. There are many things you can do with ZeroFrame, including talking to ZeroNet Plugins[^1]. A few of the most common things that you can do with ZeroFrame are:
* Open Link in New Tab
* Show Notifications and Prompts to the User
* Read and Write to Files
* Query the Database
* Sign and Publish files
* Show notifications to the User's ZeroHello NewsFeed (via the NewsFeed plugin)

In this lesson, we are only going to talk about the `onOpenWebsocket` and `onRequest` methods *and* getting the Site Info.

#### Defining Our Own Class
In order to use ZeroFrame and provide our own methods and properties, we create our own class that extends the ZeroFrame class. Before we can do this though, we must first include the `ZeroFrame.js` file.

```html
<script type="text/javascript" src="js/ZeroFrame.js"></script>
```

The class is then defined:
```javascript
class Page extends ZeroFrame {
	// ...
}
```

Afterwards, we Instantiate our class so we can access the methods and properties outside of the class:
```javascript
page = new Page();
```

Inside this class, we have three methods. The first is called `onOpenWebsocket`, and is what we will be talking about next.

#### onOpenWebsocket
The `onOpenWebsocket` method is called when your page has been loaded. This method is useful for getting site information and initializing everything on startup. Notice that ZeroHello generated code inside of this method.

```javascript
onOpenWebsocket() {
	this.cmd("siteInfo", [], function(site_info) {
		page.setSiteInfo(site_info)
	})
}
```

The `cmd` method is called whenever you want to communicate with the ZeroNet Client. You pass in the command name as the first argument. In this case, it's `siteInfo`. This command returns information on our site - including who is currently logged in and peer count.

The second argument passed in are the parameters for the command. In this case, `siteInfo` doesn't take any parameters, so we pass in an empty array.

The next argument is a callback function. This is needed because *ZeroFrame commands run asynchronously* - which means the things called after the `cmd` method **don't** wait until the command is finished. This is what the third argument is for. Once the command has been run, the function will be called with the return information as the arguments. In this case, the command returns the site information.

In this case, once the command has finished, the callback function is ran, and calls the `setSiteInfo` method with the site information passed in. This method simply displays the site information onto the page:

```javascript
setSiteInfo(site_info) {
	var out = document.getElementById("out")
	out.innerHTML =
		"Page address: " + site_info.address +
		"<br>- Peers: " + site_info.peers +
		"<br>- Size: " + site_info.settings.size +
		"<br>- Modified: " + (new Date(site_info.content.modified*1000))
}
```

Notice that we call this *inside* of the callback function. If you need to do anything that requires reading from `site_info`, you *must* put it *inside* of this callback function - otherwise there's a great chance it will run while `site_info` is still `null` or `undefined` - or whatever default value you may have set it to.

> *TIP:* You can easily see everything that `site_info` gives you by simply printing it out to the console.

#### onRequest
The next method is called `onRequest`. This method is called by ZeroNet everytime something has happened to your zite. For example, if the site information has been changed (peer count changed, zite has been updated, logged in user has changed, etc).

```javascript
onRequest(cmd, message) {
	if (cmd == "setSiteInfo")
		this.setSiteInfo(message.params)
	else
		this.log("Unknown incoming message:", cmd)
}
```

In this case, whenever the site information has been changed, we are calling `setSiteInfo`, which shows the new information on the page. Notice that we are using the `message.params` variable.

`onRequest` also sends messages with the command. This message contains a `params` property. This is what we are using to get the site information. The `params` property can also contain the `event` property, which gives you the event that happened. This is useful for doing an action on the `file_done` event. This event is called whenever a file has been updated and downloaded and is very useful for checking whether the database was updated in order to dynamically change the content on the zite.

### Requirement of the ZeroFrame Class
If you want your zite completely static, and not change based on the user logged in, a database, site information, etc. then there might not be a need for you to include the ZeroFrame class - **unless** you are using any of the ZeroFrame API calls.

There are some API calls that are useful even for static zites. For example, notifications, prompts, reading files (e.g. json files), opening a link in a new tab, client-side routing, and more. You can find more information about these and other ZeroFrame API commands in the [ZeroFrame API Docs](/1Docsfk5uLvuXkL4U13eYDxCm2gzi1fNQq/site_development/zeroframe_api_reference/index.html).

> *TIP:* All of the commands listed in the [ZeroFrame API Docs](/1Docsfk5uLvuXkL4U13eYDxCm2gzi1fNQq/site_development/zeroframe_api_reference/index.html) are always called using the `cmd()` method.

## Signing and Publishing Your Zite
You can start adding to your zite now by modifying the index page, creating new pages, etc. However, after your zite is done, you need to be sure to sign and publish it. *Signing* your zite will do a few things:
* Create a Digital Signature that is stored in the `content.json` file
* Update the `content.json` file to include the new hashes of all the non-ignored files in your zite
* Change the `modified` field in the `content.json` file

All of these are done to ensure the security of your zite. They are used so peers can reject changes to your zite that weren't made by you. The `modified` field is used by clients to ensure they are only downloading a `content.json` file that is newer than the one they have.

If you have peers, you need to send your updated zite's files to other peers so they can then send it to more peers until everyone has the updated zite. You can do this by *publishing* your zite. However, if you have no peers yet, there is no need to do this because any new visitors to your zite will just download these updated files from your client.

In order to publish and sign your zite, drag the `0` button in the top right corner to the left, scroll all the way down to the `This is my site` section. Click `Sign`, then `Publish`. This is also where you can more easily change the zite's title and description.

---
Previous Tutorial: [The Basics](./?/tutorials/the_basics)<br>
Next Tutorial: [Users and Databases In ZeroNet](./?/tutorials/users_databases_in_zeronet)

[^1]: To learn how to write a ZeroNet plugin, you can read the [Writing A Plugin tutorial](./?/write-a-plugin).
