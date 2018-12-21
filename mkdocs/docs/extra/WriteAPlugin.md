## 前言

零网可以用插件扩展功能、提供额外的站点API，实际上有些功能就是一个插件实现的，比如`MergerSite` 和 `Newsfeed`。但是零网现在还是没有插件商店，需要手动安装插件才行。不过添加、禁用、创建插件还是挺简单的。

在本教程中，你将学到如何写一个零网插件，给站点提供额外的API。

## 零网插件

零网客户端下有个`plugins`目录。如果你用的是ZeroBundle，该目录应位于`core`文件夹。

结构类似于：

``` json
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

有一些目录带有 `disabled-` 前缀，意为该插件非活跃，可以移除前缀以启用插件，再重启客户端，就能使用这些插件了。

这意味着若要添加新插件，直接把插件目录复制到 `plugins` 文件夹中即可。

## Hello World ZeroNet Plugin

为了创建新插件，需要创建新文件夹，名为`HelloWorld`。

```bash
$ mkdir HelloWorld
$ cd HelloWorld
```

再分别创建两个文件： `__init__.py` 和 `HelloWorldPlugin.py`.

```bash
$ touch __init__.py HelloWorldPlugin.py
```

在 `__init__.py` 中 import 插件。

```python
import HelloWorldPlugin
```

好玩的来了。。

创建一个API，然后站点ZeroFrame调用命令`helloworld`，插件就会返回json数据`Hello World !`。

为了实现这点，要在 `HelloWorldPlugin.py` 文件中创建 `UiWebsocketPlugin` 类。

```python
from Plugin import PluginManager

@PluginManager.registerTo("UiWebsocket")
class UiWebsocketPlugin(object):

```

`PluginManager.registerTo("UiWebsocket")` 装饰器用于注册插件，会让零网装载这个插件，并扩展了 `UiWebsocket`。

下一步，创建一个**action**，用于websocket通信。在`UiWebsocketPlugin`添加这个函数：

```python
from Plugin import PluginManager

@PluginManager.registerTo("UiWebsocket")
class UiWebsocketPlugin(object):

    # Create a new action that can be called using zeroframe api
    def actionHelloWorld(self, to):
        self.response(to, {'message':'Hello World'})
```

!!! warning
    `action`前缀是强制性的。

有两个重要参数：

1. **to**：即调用该API的站点
2. **response(to, json)**：通过websocket以json格式响应站点。

插件做好了，试一下。

!!! note
    还有其他接受插件的类，包括：

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

## Hello World 站点

为了测试插件，接下来创建一个简单的网站。

打开`index.html`：

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

添加新函数到 `Page` 用于输出信息：

```javascript
setHelloWorld(message) {
  var out = document.getElementById("out")
  out.innerHTML = message
}
```

修改 `onOpenWebsocket` 函数，不调用 `"siteInfo"` api，换成刚创建的的 `"helloWorld"` api.

```javascript
onOpenWebsocket() {
  var self = this
  this.cmd("helloWorld", [], function(response) {
    self.setHelloWorld(response.message)
  })
}
```

!!! note
    插件内`action`函数的名称后面一段决定了api名，即移除`action`前缀，并小写第一个字母。

最终代码：

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

很多分布式应用程序能作为零网插件运行，才成就了零网。零网自带一些插件，现在你可以自己写一个了。
