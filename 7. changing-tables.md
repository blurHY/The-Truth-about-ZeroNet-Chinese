# 编辑数据库

这章讲述了如何编辑 _SQLite_.

## 自增

想添加一篇文章，但 ID 用几呢？这有个蛮好的答案。先向`data/admin/data.json`添加一个新属性。

```json
{
	"next_post_id": 3,
	"posts": [
		{
			"id": 1,
			"title": "Post 1",
			"content": "Post 1"
		},
		{
			"id": 2,
			"title": "Post 2",
			"content": "Post 2"
		}
	]
}
```

许多站点都这么做: 使用 `next_..._id`. 咱也这样. 添加新文章时，以`next_post_id`为新 ID。

## 数据库的写权限

之前已经说过了, 只有 _SELECT_ 是允许的。还记得咱从哪编辑文章的吗?嗯对，在 `data/admin/data.json` 文件里。就用上节课讲的 `readFile()` 和 `writeFile()` 函数:

```js
function readFile(file, callback) {
	zeroFrame.cmd("fileGet", [file, false], callback);
}

function writeFile(file, content, callback) {
	zeroFrame.cmd("fileWrite", [file, base64Encode(content)], callback);
}

function base64Encode(content) {
	content = encodeURIComponent(content); // Split to bytes in % notation
	content = unescape(content); // Join % notation as bytes (not as chars)
	return btoa(content);
}
```

添入 `js/files.js`. 别忘了把这文件也引入到 `index.html` ，而且要放在在 `index.js`之前：

```html
<script type="text/javascript" src="js/files.js"></script>
```

第一步要读取 `data/admin/data.json`文件并解析，试着自己弄下。

答案:

```js
function addPost(title, postContent) {
	readFile("data/admin/data.json", function(content) {
		content = content || ""; // Convert undefined and null to ""

		// Parse JSON
		try {
			content = JSON.parse(content);
		} catch (e) {
			content = {
				posts: [],
				next_post_id: 0,
			};
		}

		console.log(content);
	});
}

addPost("test", "test content");
```

打开控制台并刷新页面，就能看到有 `posts` 和 `next_post_id` 属性的对象。

简单地用 `next_post_id` 并编辑 `posts` 数组:

```js
    function addPost(title, postContent, callback) { // Notice new callback argument
        readFile("data/admin/data.json", function(content) {
            ...
            // Parse JSON
            ...

            content.posts.push({
                id: content.next_post_id++, // Use next_post_id and then increment it
                title: title,
                content: postContent
            });

            content = JSON.stringify(content, null, 4); // Make content string

            writeFile("data/admin/data.json", content, function(result) {
                callback(result == "ok");
            });
        });
    }

    addPost("test", "test content", function(result) {
        console.log(result ? "OK" : "Error");
    });
```

尝试运行代码，观察 `data/admin/data.json` 的变化. 打开 `data/mydatabase.db`. 咋只有以前的文章?因为改完数据还没有签名。下面是我们需要的 API:

### siteSign

给站点的content.json签名

| 参数                               | 定义                                                                   |
|------------------------------------|------------------------------------------------------------------------|
| **privatekey** (可选)              | 用于签名的私钥 (默认值: 当前用户的私钥)                                |
| **inner_path** (可选)              | 欲签名的content json文件的内部路径 (默认值: content.json)              |
| **remove_missing_optional** (可选) | 移除在content.json文件内，但实际上已不存在的可选文件。 (默认值: False) |

**返回值** 成功则返回"ok" ，失败则返回错误信息。

> **注意：**
> 如果私钥在users.json中，就用"stored"作为privatekey的参数值。(例如 cloned sites)

### sitePublish
发布一个站点的content.json

| 参数                  | 定义                                                    |
|-----------------------|---------------------------------------------------------|
| **privatekey** (可选) | 用于签名的私钥 (默认值: current user's privatekey)      |
| **inner_path** (可选) | 欲发布的content json的内部路径 (默认值: content.json)   |
| **sign** (可选)       | 若值为True则会在发布前给content.json签名 (默认值: True) |

**返回值** "ok" on success else the error message

```js
    writeFile("data/admin/data.json", content, function(result) {
        if(result != "ok") {
            callback(false);
            return;
        }

        zeroFrame.cmd("sitePublish", {
            privatekey: "stored"
        }, function(result) {
            callback(result == "ok");
        });
    });
```

试着刷新页面并打开数据库 `data/mydatabase.db`.

## 私钥

 `siteSign` 和 `sitePublish` 命令中主要是 `privatekey` 参数，这个参数比较难懂，但经常用。你得记住它的意思。

-   `privatekey: null` 意为 "使用零网为用户和站点创建的私钥". 有些站点(like ZeroTalk) 包含用户内容 (帖子和投票). 使用用户的密钥签名数据，并不用传递`privatekey`参数，直接设为`null`.
-   `privatekey: "stored"` 意为 "使用站点的私钥" 只有在用户是站长时才能用。
-   `privatekey: "..."` 意为 "使用指定的私钥".

## 权限

我们也经常需要知道用户是否为站长，例如，博客，只应该向站长显示“添加文章”按钮。ZeroFrame 就是! 利用`siteInfo` 命令， 它返回一个js对象，包含一个 `privatekey` 的布尔值属性 - 如果是true那么当前用户就是站长。

```js
    function isAdmin(callback) {
        zeroFrame.cmd("siteInfo", [], function(info) {
            callback(!!info.privatekey);
        });
    }
```

## 例子

 [博客例子](downloads/blog.html).

改进建议:

-   添加一个删除按钮。
-   更好看的编辑页面。
-   在`Add post`页面，`Cancel` 点击后删除创建的文章
-   使用非HTML的标记语言
