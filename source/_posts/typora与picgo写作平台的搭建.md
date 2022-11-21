---
title: Typora+PicGo写作平台的搭建
hide: false
date: 2022-01-03 12:57:03
excerpt: Typora写作通常会插入图片，一个简单的方式是准备单独的图片文件夹，这是特异性的解决方式，治标不治本。如何做到文章只写一遍，图片只保留一份？这时候我们就需要图床了。接下来我会介绍Typora与图床的一条龙写作服务。
tags:
  - 图床
  - Makrdown
categories:
  - 博客
---

> 前前言：
>
> 2022 年 03 月 24 日
>
> 这天我的  Gitee 图床被屏蔽了，所以后面的这些教程，把它看作  GitHub 就行了。如果真个想使用  Gitee ，建议自己的图片做个备份，经常换仓库。不然要死直接全死。这里也要说明一下，写了封邮件之后，16 个小时左右仓库就解封了。不过解封归解封， Gitee 再见！
>
> 子曰：资本家的羊毛不可薅。

# 前言

 Typora 是优秀的本地写作软件，但是我们的文章可能不仅仅是保留在本地。有些时候我们会进行数据迁移，例如换一台电脑，或者发表到网上，我们会希望自己的图片只有一份，因为这样迁移的麻烦少了一个。图床是解决图片问题的最好方式。

> 所谓 图床 ，就是把你的图片放到一个固定的地方(常指网络)，然后通过连接访问，这样要使用图片的时候，只需要知道图片的链接就行，不需要再保留一份。在   Markdown 写作中我们知道，使用图片的方式是`[图注](链接)`。因此，当用 Markdown 进行写作的时候，我建议配合图床使用——把图片保留在网络上而不是本地。

现在可选的的图床很多，免费的、不免费的、国内的、国外的，但稳定的、普适的图床却很少。我用过的有 sm.ms 图床和  Gitee 。鉴于例如公众号这类平台有屏蔽外链的风险，我选择长期使用  Gitee 作为图床。  Gitee 是国内最大的代码托管平台，内部提供仓库 2g (2021年)。这存储空间够用了。当然，  GitHub 也可以作为选择，但是访问可能慢一点，而且空间也少一点。

闲话少叙，准备好工具：

-  Gitee [^1]账号
-  Typora [^2]编辑器，Markdown 写作
-  GicGo [^3]图床配置工具

以上所需的工具都是免费的。

---

# PicGo准备

 PicGo 是开源免费的图床配置工具，我一直在用。官网的介绍很清楚，如果有不清楚的可以看官网的文档。

![下载PicGo](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051328651.png)

点击下载之后一直往下拉，找到最新版的下载软件，

![找到最新版(就是最上面的一版)](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051335431.png)

![下载指定版本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051337609.png)

下载之后就是简单的安装，安装完之后打开。

>  注意： 
>
>  PicGo 打开的时候只在任务栏出现，所以去任务栏点击打开。

![打开PicGo](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051338760.png)

因为 PicGo 没有内置  Gitee ，所以去插件设置查找  Gitee 的插件安装。这三种都差不多，选择第一个就像我这样。

![查找插件并下载](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051339134.png)

插件安装的时候，系统会提示你安装 Node.js ，同意安装就行了。安装之后效果如下：

![安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051756804.gif)



![安装node.js](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051755069.png)

![安装插件效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051344742.png)

好了， PicGo 准备好了，接下来准备白嫖  Gitee 的 2G 空间。

---

#  Gitee账号

以下为申请  Gitee 账号，如已有，请跳至 仓库配置 。

## 申请账号

首先去  Gitee 官方网站，点击注册。注册使用的是邮箱，可以临时申请一个邮箱——但是必须可以接受消息，因为后面需要验证。

![官网注册](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051319152.png)

![注册](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051350833.png)

注册时需要用邮箱接收验证码，完成之后，登录进入主界面，创建新仓库：

![创建仓库](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051352300.png)

![创建仓库](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051354452.png)

创建完成。

---

## 仓库配置

为了让 PicGo 行使生杀大权，我们需要给它一个令牌( token )，大概类似于一种权限。

![打开设置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051359935.png)

![私人令牌](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051401471.png)



![生成新的令牌](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051401541.png)

![描述可以写上一些信息](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051402140.png)

注意，令牌生成的时候复制下来保管好，因为它只生成一次，忘了就得重新申请。这里我就不重新申请了。把令牌粘贴到 PicGo 的  Gitee 图床的 token 框。

![粘贴令牌](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051405605.png)

![看图写话](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051410924.png)

然后设置为默认图床，确定，万事大吉！！

---

### 配置Typora

默认已经安装了 Typora 了。

![配置Typora](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201051415186.png)

至此，你完成了所有的配置。点击 验证图片上传选项 验证是否成功。如果出现问题，最好检查你的 PicGo 中的 repo 写对没有。

# 验证写作

我们验证一下效果：

![上传图片](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202201031746951.gif "上传图片")

完成！

---

# 后记

借助图床，   Markdown 式的写作更加轻松，也节约了图片管理的时间。然而需要注意的是， PicGo 上传的图片大小最好不要超过 1M ，因为这会影响加载的速度。而且有些时候，你的图片可能会被  Gitee 吞掉，原因不明，也许是版权问题。

> PicGo 和 Typora 的安装包提供(无需登录，直接复制链接输入提取码即可):
> 链接: https://www.123pan.com/s/gqA9-llYB	提取码:csym



# 相关网址

[^1]: https://Gitee.com/
[^2]: https://Typora.io/
[^3]: https://PicGo.GitHub.io/PicGo-Doc/zh/