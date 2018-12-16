# ZeroFrame

前面那章我说了 _localStorage_ 等特性被禁用了。在这章，我将讲述如何不使用那些特性实现相应功能。

## WebSockets

为了让站点能读写文件、使用 _localStorage_、给数据签名，等。零网使用 WebSockets，_ZeroFrame_ 已经做好相关准备工作。记得 `js/ZeroFrame.js`吗? 这个就是 API 接口。

## 安装 ZeroFrame

创建新站点，打开站点目录

把 `index.html` 改为:

```hmtl
    <!DOCTYPE html>
    <html>
        <head>
            <title></title>
            <meta charset="utf-8">
            <meta http-equiv="content-type" content="text/html; charset=utf-8" />
            <base href="" target="_top" id="base">
            <script>
                base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")
            </script>
        </head>
        <body>
        </body>
    </html>
```

我已经移除 `<script>`并缩进代码。

引入 `js/ZeroFrame.js` 并创建文件 `js/index.js`:

```html
<!DOCTYPE html>
<html>
	<head>
		<title></title>
		<meta charset="utf-8" />
		<meta http-equiv="content-type" content="text/html; charset=utf-8" />
		<base href="" target="_top" id="base" />
		<script>
			base.href = document.location.href
				.replace("/media", "")
				.replace("index.html", "")
				.replace(/[&?]wrapper=False/, "")
				.replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "");
		</script>
	</head>
	<body>
		<script type="text/javascript" src="js/ZeroFrame.js"></script>
		<script type="text/javascript" src="js/index.js"></script>
	</body>
</html>
```

打开、编辑 `js/index.js`:

```js
var zeroFrame = new ZeroFrame();
console.log(zeroFrame);
```

打开开发者工具并刷新页面：

```json
    Object { url: undefined, waiting_cb: Object, wrapper_nonce: "45be457a20334e09dbda338264976328d1f…", target: Object → 1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm, next_message_id: 1 }
```

可见类似如上的信息。

这就是 _ZeroFrame_ —— 接收、发送器，零网站点的基本框架之一。

## 首次调用 API

ZeroFrame 的主要函数为 `ZeroFrame.prototype.cmd()`. 试下获取站点信息

```js
var zeroFrame = new ZeroFrame();
zeroFrame.cmd("siteInfo", [], function(info) {
	console.log(info);
});
```

第一行尚能看懂。顺便说下，不要创建多个 ZeroFrame 实例。剩下几行就是调用 API，第一个参数是要调用的方法 - 这段代码里是 `siteInfo`，第二个参数是传递给方法的参数 - 是可选的，所以可以移除 `, []` 第三个参数就是回调。

刷新页面，观察控制台，可见：

```json
    Object { tasks: 0, size_limit: 10, address: "1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm", next_size_limit: 10, auth_address: "1E3sYXHq99vXW5hS3M9PhQzQhpmWx1paHu", feed_follow_num: null, content: Object, peers: 1, auth_key: "db87a590a113015ddf1b2c73f96baacaa097eee1747d71bdbe5748073bc6131b", settings: Object, 6 more... }
```

有个问题就是 ZeroFrame 使用回调，会引起回调地狱。我认为 Promise 更好，此外，Promise 兼容 async/await。如果你要用回调，就使用 ZeroFrame。如果你想用 Promise 或 async/await (零网不支持后者，但你可以用 Babel)，用我写的*ZeroPage library* (额外课程里讲了 ZeroPage，在[这](downloads/ZeroPage.js)下载. 我会教你怎么写的 [here](downloads/notepad.html).

新版 ZeroFrame 有`ZeroFrame.prototype.cmdp()` 函数了，它返回一个 Promise。但我仍然推荐我的库，这样你可以少写代码。

## 文件系统

ZeroFrame 让你能编辑站点目录内的文件，API 定义为:

fileGet - 
返回文件内容或者 null、undefined。
|        参数         |                定义                 |
|:-------------------:|:-----------------------------------:|
|     inner_path      |       你需要的文件的内部路径        |
| required (optional) |    是否等待文件下载，默认为 True    |
|  format (optional)  | 编码，text 还是 base64，默认为 text |
| timeout (optional)  |      最大等待时间，默认为 300       |

fileWrite - 
返回"ok"或者错误信息。
|      参数      |              定义              |
|:--------------:|:------------------------------:|
|   inner_path   |     你需要的文件的内部路径     |
| content_base64 | 要写入的内容（先编码成base64） |


如果你用 _ZeroPage_ 库，我也有 _ZeroFS_，[这里](downloads/ZeroFS.js)下载

编辑`index.js`为：

```js
var zeroFrame = new ZeroFrame();

function readFile(file, callback) {
    ...
}
function writeFile(file, content, callback) {
    ...
}
```

### 读取文件

看答案之前，试试自己用 `readFile()`，准备好了吗?

```js
function readFile(file, callback) {
    zeroFrame.cmd("fileGet", [file, false], callback);
}
```

也可以写成：
```js
function readFile(file, callback) {
    zeroFrame.cmd("fileGet", {
        inner_path: file,
        required: false
    }, callback);
}
```

参数可以用数组传递也可以用对象传递，详见文档。

若 `required` 设为 `true`，代码仍能运行，就是会有一个警告。

### 写入文件

`writeFile()` 很复杂，我想说，这太难用了。输入的得是Base64，所以你要写：

```js
function writeFile(file, content, callback) {
    zeroFrame.cmd("fileWrite", [file, btoa(content)], callback);
}
```

但 `btoa()` 不能用于 UTF8，所以又要写另一个函数：
```js
function base64Encode(content) {
    content = encodeURIComponent(content); // Split to bytes in % notation
    content = unescape(content); // Join % notation as bytes (not as chars)
    return btoa(content);
}
```

然后替代`writeFile()`里的 `btoa()`。

我们转换UTF8到字节数据，但如果 `content` 就是字节数据呢? 
`encodeURIComponent`和`unescape`就会破坏数据，因为Unicode一个字符使用多个字节。所以我们用另一个函数，其中使用 `btoa()` 而非 `base64Encode` (按 `writeFile()` 你自己写 `writeBinaryFile()`).

文件系统的库已经写好，不过只有文件操作的部分，但已经足够了。你可以读我的 _ZeroFS_ 理解零网的文件操作，或者读零网的文档 [镜像](/1HP65eGEEMPbyzH3mkaoQ7eCMKd9P8G61W/).

## 记事本

用文件操作的API的最简单的程序可能就是编辑器了，试着自己写一个，再尝试读懂我的代码 [notepad](download/notepad.html)。

## 小提示

零网还是有很多问题的，有些问题根本找不出来原因（我花了两个月调试数据库——因为零网缺少文档，只能读源码），你可以随便问我问题 [ZeroMail](/1MaiL5gfBM1cyb4a8e3iiL8L5gXmoAJu27/?to=ivanq).

> 译者注：你联系我也可以，blurhy@outlook.com
