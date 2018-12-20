## Introduction

In ZeroNet, plugins can be created to extend the functionality of the client as well as provide api calls to sites. Some of the features that we use are actually plugins that can be downloaded, like `MergerSite` and `Newsfeed`. There currently isn't a plugin store where you can install them. However, it is simple to add, disable, or create plugins. In this tutorial, you will learn how to write a simple plugin that provides an api which can be called directly from a site.

> **Note**: I advise you to read [The Basics](/14pM9huTYzJdyQyHRj6v2kfhMe8DrxwpGt/?/tutorials/the_basics) tutorial first. You should also know to create a site.


## ZeroNet Plugins

In your ZeroNet folder will find a `plugins` folder. If you have installed ZeroNet through ZeroBundle, you might find it under the `core` folder.

You should see a bunch of folders like the following:

```
.
├── AnnounceZero
├── Cors
├── CryptMessage
├── disabled-Bootstrapper
├── disabled-Dnschain
├── disabled-DonationMessage
├── disabled-Multiuser
├── disabled-StemPort
├── disabled-UiPassword
├── disabled-Zeroname-local
├── FilePack
├── MergerSite
├── Mute
├── Newsfeed
├── OptionalManager
├── PeerDb
├── Sidebar
├── Stats
├── TranslateSite
├── Trayicon
└── Zeroname

```

Notice that some folders have the prefix `disabled-`. This means that those plugins are not active. You can activate them by renaming the folder so that the prefix is removed. Once ZeroNet restarts, those plugins will load with the rest of them.

This means that if you want to add a new plugin, you just need to copy it into a new folder inside this `plugins` folder.

## Hello World ZeroNet Plugin

In order to start creating our new ZeroNet Plugin, we will need to create a new folder with the name of our plugin: `HelloWorld`.

```bash
$ mkdir HelloWorld
$ cd HelloWorld
```

Now, we need to create two seperate python files: `__init__.py` and `HelloWorldPlugin.py`.

```bash
$ touch __init__.py HelloWorldPlugin.py
```

In `__init__.py`, we simply need to import our plugin.

```python
import HelloWorldPlugin
```

Now we are getting to the interesting part...


Lets create a new zeroframe api call that will allow us to comunicate with a site using the command `helloWorld` which will return a json message that says `Hello World !`.

To do this, we need to create a `UiWebsocketPlugin` class in our `HelloWorldPlugin.py` file.

```python
from Plugin import PluginManager

@PluginManager.registerTo("UiWebsocket")
class UiWebsocketPlugin(object):

```

Notice the `PluginManager.registerTo("UiWebsocket")` decorator. It registers and loads our plugin into ZeroNet and extends our class as a `UiWebsocket`.

Next, we need to create an _action_ to communicate our message via websocket. Let's define it in our class `UiWebsocketPlugin`...

```python
from Plugin import PluginManager

@PluginManager.registerTo("UiWebsocket")
class UiWebsocketPlugin(object):

    # Create a new action that can be called using zeroframe api
    def actionHelloWorld(self, to):
        self.response(to, {'message':'Hello World'})
```

We have added an `actionHelloWorld` method to our class. Notice here that the `action` prefix is mandatory in order to be called through the zeroframe api.

We have two important elements that we are using here:
1. **to**: represents the site that called our command.
2. **response(to, json)**: the method that returns the response to the site through websocket using json format.

We have finished our plugin. Now, let's test it!

> **Note** : Here, I am only showing how to create a UIWebsocket plugin, but there are other classes that accept plugins. These include:
- UiRequest
- User
- UserManager
- WorkerManager
- TorManager
- Site
- SiteManager
- FileRequest
- ContentDb
- ConfigPlugin
- Actions
- SiteStorage
- UIWebsocket

## Hello World site

In order to test out our new plugin, we need to create a site. This site is going to be fairly simple. You can find out how to create a simple site by following the other tutorials on the Dev Center, as well as on [ZeroBlog](/Blog.ZeroNetwork.bit/?Post:99:ZeroChat+tutorial+new).

![Create New site](./img/create-new-site.png "Create New Site")

Click on `create new site` on the ZeroHello page. Once you have done that, open directory where your site files are stored. You should have these files in your site folder:

```
.
├── content.json
├── index.html
└── js
    └── ZeroFrame.js

1 directory, 3 files
```

Open the `index.html` file. You should have the following lines of code:

```javascript
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
```

Let's add a new method to our `Page` class that will print our message to the page. We will call it `setHelloWorld`.

```javascript
setHelloWorld(message) {
  var out = document.getElementById("out")
  out.innerHTML = message
}
```

We are then going to modify the `onOpenWebsocket` method so that instead of calling the `"siteInfo"` api, we will call our `"helloWorld"` api.

```javascript
onOpenWebsocket() {
  var self = this
  this.cmd("helloWorld", [], function(response) {
    self.setHelloWorld(response.message)
  })
}
```

Now all we have to do is pass the message from the response into our `setHelloWorld` method so that it can modify the html dom and show our message.

> **Note**: The name of our action method in our plugin will give us the name of our api. It should be noted that we have removed the `action` prefix and the capital letter on the first word.

Final code:

```javascript
class Page extends ZeroFrame {
	setSiteInfo(site_info) {
		var out = document.getElementById("out")
		out.innerHTML =
			"Page address: " + site_info.address +
			"<br>- Peers: " + site_info.peers +
			"<br>- Size: " + site_info.settings.size +
			"<br>- Modified: " + (new Date(site_info.content.modified*1000))
	}



  setHelloWorld(message) {
    var out = document.getElementById("out")
    out.innerHTML = message
  }

	onOpenWebsocket() {
    var self = this
		this.cmd("helloWorld", [], function(response) {
      self.setHelloWorld(response.message)
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
```

There are many distributed applications which we could plug into Zeronet. This is why ZeroNet is such a great application. There are already some plugins, and now you can start to write your own also.
