# 聚合站点 (Merger sites)

看下ZeroMe，一个有一万用户的社交网络，每个用户都有很多数据，总共大概600MB。很大，不是很好，好在零网引入了聚合站点以解决这个问题。


## 结构

想象一下下面的结构：

```py
ZeroMe = 500 MB
|
+-- data = 500 MB
|-----------------------------------------------------------------------------------|
| +-- users = 500 MB                                                                |
|                                                                                   |
| +-- user1 = 50 KB                                                                 |
| +-- user2 = 50 KB                                                                 |
| +-- user3 = 50 KB                                                                 |
| +-- user4 = 50 KB                                                                 |
| +-- user5 = 50 KB                                                                 |
| +-- user...                                                                       |
| +-- user10000 = 50 KB | 50 KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB
+-- index.js = 250 KB
+-- index.css = 100 KB
+-- index.html = 50 KB
```

这样的话用户就得下载其他用户的所有数据 (`userN`目录).

这样呢:

```py
ZeroMe = 400 KB
|
+-- index.js = 250 KB
+-- index.css = 100 KB
+-- index.html = 50 KB

UserDB = 10 MB
|
+-- data = 10 MB
|----------------------------------------------------------------------------------|
| +-- userinfo = 10 MB                                                             |
|                                                                                  |
| +-- user1 = 1 KB                                                                 |
| +-- user2 = 1 KB                                                                 |
| +-- user3 = 1 KB                                                                 |
| +-- user4 = 1 KB                                                                 |
| +-- user5 = 1 KB                                                                 |
| +-- user...                                                                      |
| +-- user10000 = 1 KB |= 1 KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB
+-- index.html = 5 KB

Hub1 = 167 MB
|
+-- data = 167 MB
|---------------------------------------------------------------------------------|
| +-- users = 167 MB                                                              |
|                                                                                 |
| +-- user1 = 50KB                                                                |
| +-- user2 = 50KB                                                                |
| +-- user...                                                                     |
| +-- user3333 = 50KB |= 50KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB
+-- index.html = 5 KB

Hub2 = 167 MB
|
+-- data = 167 MB
|---------------------------------------------------------------------------------|
| +-- users = 167 MB                                                              |
|                                                                                 |
| +-- user3334 = 50KB                                                             |
| +-- user3335 = 50KB                                                             |
| +-- user...                                                                     |
| +-- user6666 = 50KB |= 50KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB
+-- index.html = 5 KB

Hub3 = 167 MB
|
+-- data = 167 MB
|----------------------------------------------------------------------------------|
| +-- users = 167 MB                                                               |
|                                                                                  |
| +-- user6667 = 50KB                                                              |
| +-- user6668 = 50KB                                                              |
| +-- user...                                                                      |
| +-- user10000 = 50KB |= 50KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB |KB
+-- index.html = 5 KB
```

所以我们就把数据分割入不同站点，聚合站就是这样。 _ZeroMe_ (`1MeFqFfFFGQfa1J3gJyYYUvb5Lksczq7nH` ) 只有网页和代码, _UserDB_ (`1UDbADib99KE9d3qZ87NqJF2QLTHmMkoV`) 用户元数据,其他站点 - _hubs_ 有其他数据。当然，可以不下载这些hub站点，都是可选的。

例如，若你想下载 `ZeroMe`, `UserDB` 和 `Hub1`, 你可以联系 `user878` 和 `user678` 并得知那还有 `user5412`, `user3453` 和 `user6789` 都在 `Hub2` 和 `Hub3` 站点. 要想向 `user6789` 打招呼或者读他的帖子，得先下载 `Hub3`. 但通常你只需要读一部分帖子，并不是全网的帖子。所以下载一两个Hub够了。

## 创建聚合站点

先创建新站点，打开侧边栏，命名站点为 `PostHere`. 移除`index.html`中 `<body>`元素里的所有内容，用户数据存在Hub里，代码放在 `PostHere`站点.

## 创建 Hub

创建一个新站点存储用户内容，我创建的站点地址为 `1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ` 。

把 `index.html` 改为:

    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8">
            <meta http-equiv="content-type" content="text/html; charset=utf-8" />
            <base href="" target="_top" id="base">
            <script>
                base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")
            </script>
        </head>
        <body>
            Use PostHere to watch content of this site.
        </body>
    </html>

改标题并签名 `content.json`.

## 配置聚合站点

聚合站点 (在我这 `1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm`) 和 被聚合站点 (`1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ`) 需要互相同意才能建立链接。

所以把下列内容添加到 被聚合站点 (`1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ`)的根 `content.json`

    "merged_type": "PostHere",

创建目录 `merged-PostHere` 于聚合站点的根目录，让用户给这个目录签名，就添加:

    "ignore": "merged-.*",

添加到主站点的 `content.json`，还需要请求`Merger:PostHere`的许可:

    +-------------------------------------------------------------------------+
    |                          wrapperPermissionAdd                           |
    +-------------------------------------------------------------------------+
    | Request new permission for site                                         |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | permission              | Name of permission                            |
    +-------------------------+-----------------------------------------------+
    | Return: "Granted" if allowed, not send when disallowed                  |
    +-------------------------------------------------------------------------+

So, add the following to `js/index.js` file and add it to `index.html` of PostHere:

    window.zeroFrame = new ZeroFrame();

    function requestPermission(permission, callback) {
        zeroFrame.cmd("siteInfo", [], function(siteInfo) {
            // Already have permission
            if(siteInfo.settings.permissions.indexOf(permission) > -1) {
                callback();
                return;
            }

            zeroFrame.cmd("wrapperPermissionAdd", [permission], callback);
        });
    }

    requestPermission("Merger:PostHere", function() {
        // TODO
    });

## 访问聚合站

ZeroNet doesn't provide us a way to read and write other site's content, so MergerSite plugin makes all `merged-...` directories virtual. So to ZeroNet `merged-PostHere` structure is:

    merged-PostHere
    |
    +-- merger.db
    +-- 1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ
        |
        +-- js
|----------------------------------------------------------------------------------|
| +-- ZeroFrame.js |roFrame.js |js |js |js |js |js |js |js |js |js |js |js |js |js |js
        +-- data
|--------------------------------------------------------------------------------------|
| +-- users                                                                            |
|                                                                                      |
| +-- user1                                                                            |
| |                                                                                    |
| |   +-- data.json                                                                    |
| |   +-- content.json                                                                 |
| +-- content.json     |ntent.json |on |on |on |on |on |on |on |on |on |on |on |on |on |on
        +-- index.html

We have set `db_path` to `merged-PostHere/merger.db` because ZeroNet can gather data only from files which are in database directory, which is in this case `merged-PostHere`.

So, if we want to access `data/users/{address}/data.json` of `1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ`, we have to access `merged-PostHere/1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ/data/users/{address}/data.json`.

In the following part of the tutorial, we will fill our `PostHere` zite with user content.
