# 创建站点

## 克隆站点

咱建个自己的博客吧访问 _ZeroHello_，并下载*ZeroBlog*，点击三个点的那个按钮，会弹出一个菜单：

-   收藏
-   更新
-   检查文件
-   暂停
-   克隆
-   Save as .zip
-   删除

点击**克隆**，然后博客就有了，克隆就这些。

### 更改 ZeroBlog

鼠标悬浮在博客标题、简介……上，再点击 🖊，就可以编辑了。

那么就发电啥吧，点击**添加新文章**。编辑标题、内容，再点底部的 _Sign & Publish new content_ 。对数据签名后，`content.json`的内容会改变。发布内容时，零网客户端会将站点发布到其他节点，然后他们就再分发这些内容。

要是只有你有这个站点，就会很难发布。

## 下载并更新站点

下载网站时，零网会在节点列表里找有这个网站的节点。可以是你的第二台电脑，不知道你第一台电脑的存在，若是它想下载这个站点，怎么办呢？

答案是... 用 _tracker_（**跟踪器**）所有的节点都会频繁连接 tracker 并告诉他：我的 ip 是 xx 我有这些站点 : `...`。所以呢，第一台电脑告诉跟踪器我有站点 a，然后第二天电脑问跟踪器有没有站点 a。就这么简单。

再说点关于跟踪器的，若是节点不接收你的更新（例如，它们没有这个站点），你可能要一个固定 IP 或是 Tor。

## 代理

只要有一个电脑开着（有你的网站的），网站就是上线的。而你可能想在晚上关电脑，所以有必要架设一个一直开机的电脑。

好在零网有这些电脑 _ZeroNet proxies_ （**零网代理/网门**）. [0net.io](https://0net.io)是其中之一，用法和客户端一样。那个代理上的 *ZeroTalk*地址为`https://0net.io/1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT`

做好博客之后，访问 `https://0net.io/{站点公钥（地址）}/`， 地址类似于本地客户端的 `http://127.0.0.1:43110/{站点公钥（地址）}/`.

可用代理： [0net.io](https://0net.io), [@amorgan's proxy](http://zn.amorgan.xyz) 以及 [ZeroGate](https://zerogate.tk).

## .bit 地址

站点可以通过公钥访问 (例如 `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT` 是 _ZeroTalk_)。部署这种网站不花钱，但你可能并不觉得 `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT`这名字多好，若是 `zerotalk.com` 岂不更好?

实际上，能通过 _Namecoin_ 实现。 _Namecoin_ 基于区块链。若想注册`.bit` 地址 (需要钱)，应下载 _Namecoin_ 客户端，并输入 `.bit` 地址 (只支持 `.bit` 后缀).

若是不想下载*Namecoin*客户端和整个区块链，也可以使用在线服务, 例如 Dotbit.me 和 Peername.

大多数版本的零网客户端都有 _Namecoin_ 插件，所以就能访问 _ZeroTalk_ 用.bit 地址 `http://127.0.0.1:43110/Talk.ZeroNetwork.bit/` 了。

可以在[ZeroName](http://127.0.0.1:43110/zeroname.bit/) 查看你注册的域名 `.bit`

## 练习

尝试克隆 _ZeroTalk_ 并开发一个自己的站点。
