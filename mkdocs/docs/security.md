# 安全

这章讲零网安全。如果不太感兴趣或者没法理解，稍微看看就好。

-   每个站点都有公钥和私钥。输入 `http://127.0.0.1:43110/1HeLLo4uzjaLetFx6NH3PMwFP3qbRbTf3D/` 访问 ZeroHello - 其中， `1HeLLo4uzjaLetFx6NH3PMwFP3qbRbTf3D` 为公钥。只有站长有私钥，用来发布内容。
-   下载站点时，零网会连接已知节点并向它们索要文件。
-   没有私钥就无法更改网站内容。

## 错误的 SHA512

零网中，每个人都为其所访问过的站点服务。例如, `Windows` 上有 `ZeroMe`, `iPad` 想要访问这个站点， `iPad` 就会直接 _连接_ `Windows` 并说：

-   **iPad**: 嘿, _Windows_, 给我 `ZeroMe`的 `content.json`
-   **Windows**: 嗯，给你
-   **iPad**: 谢啦，我还要 `index.html`, `js/all.js` 以及 `ZeroMe` 的 `css/all.css`
-   **Windows**: 给你 `index.html`: ... 但是我没有 `js/all.js` 和 `css/all.css`, 抱歉。

_Windows_ 给了 _iPad_ `index.html`, 但 _iPad_ 去哪找 `js/all.js` 和 `css/all.css`呢？从 _Windows_ 所知的节点？但要是 _Windows_ 怀有恶意，会提供错误的节点。

-   **iPad**: 再见，_Windows_.
-   **iPad**: ...寻找节点中...
-   **iPad**: 嘿, _MacOS_, 你有 `js/all.js` 和 `ZeroMe` 的 `css/all.css` 吗
-   **MacOS**: 当然有，给你

但 _MacOS_ 是个黑客! 他给了错误的 `js/all.js`，然后 `Windows` 就有了个带病毒的 `js/all.js`!

...实际上并非如此。每个站点都有相应的公钥和私钥

`1BewKAyyiMZHY3AjQn65J6f6Rcb9p1h64K`. 这是一个公钥，即站点地址

每个一个站点的`content.json`中，都存有站点文件的 SHA512 和大小。 **MacOS** 给了 **iPad** 带 SHA512 的文件， **iPad** 就说：

-   **iPad**: `css/all.css` 有效， `js/all.js` 被篡改。
-   **iPad**: ...添加 _MacOS_ 到黑名单...
-   **iPad**: 拜拜， _MacOS_.
-   **iPad**: ...寻找节点中...
-   **iPad**: 嘿, _Linux_, 你有 `ZeroMe`的`js/all.js` 吗 ?
-   **Linux**: 有，给你。
-   **iPad**: 谢啦，拜拜， _Linux_.

现在 _iPad_ 有完整的 _ZeroMe_ 可以向你展现了。

零网就是这样使用 SHA512 确保数据的安全。

## 错误的`content.json`

_Linux_ 需要 _ZeroTalk_ (公钥为 `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT`):

-   **Linux**: 你有 `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT` 的 `content.json` 吗
-   **MacOS**: 有，拿去吧。

正如我们所知 _MacOS_ 是个黑客，提供错误的`content.json`，SHA512 不正确。 _Linux_ 向 _MacOS_ 要 `js/all.js`，然后 _MacOS_ 会给 _Linux_ 带毒文件。

-   **Linux**: 谢谢! ...等下 `content.json` 的签名有误。
-   **Linux**: ...添加 _MacOS_ 到黑名单..
-   **Linux**: 再见， _MacOS_.

_Linux_ 很聪明， `content.json` 经过私钥签名。 _MacOS_ 可能会偷取私钥或者暴力穷举破解，后者根本不可能。前者还是可能的，所以不要让私钥被窃取。

## 签名

咱知道数据是经过签名的。`content.json` 自身被签名，其他文件的 SHA512 又是放在 `content.json` 里的。这表明没有私钥就无法改动数据，常规的彩虹表也无能为力。

## 沙盒

零网不会给每个站点都架设一个服务器，只放一个服务器在 `http://127.0.0.1:43110/`。所以还需要其他安全措施，每个站点都是放在沙箱 `<iframe>` 里面的，然后 _localStorage_ 等东西就没法用了。

零网也不会让站点逃逸出 `<iframe>` - 会传递一个特殊的口令(token)到 `<iframe>` 中，站点用口令与零网服务器通讯以便执行一些底层指令。刷新页面时口令也会便，只有暴力穷举 `wrapper_nonce` (口令 token) 才能逃逸出 `<iframe>`.

数据怎么传进 `<iframe>` 呢? 只有查询字符串（query string）所以呢 `wrapper_nonce` 就是放在查询字符串里。你可能觉得查询字串是 `?a=2`，但实际上 `?a=2&wrapper_nonce=e0ccdebc9a804cfd8ac9eaf78a4a03054acbd400ce981219037647922219fbcd`.
