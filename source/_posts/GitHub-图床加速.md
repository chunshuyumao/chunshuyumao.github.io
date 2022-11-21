---
title: GitHub 图床加速
hide: false
tags:
  - Fluid
  - 图床
categories:
  - 博客
math: false
mermaid: false
date: 2022-05-23 18:23:50
index_img:
excerpt: 为自己的 Hexo Fluid 主题注入自动选择 CDN 加速的脚本，同时记录 GitHub 图床加速。
---

# 前言

图床迁移到 GitHub 之后，我也面临 DNS 污染的问题。图片保存在 GitHub 通常不会有什么问题，就是访问比较慢，国内如此。想要加速，我们就的使用 CDN 加速。

CDN<ruby>（内容分发网络）<rt>Content Delivery Network</rt></ruby> 尽可能避开互联网上有可能影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。一句话总结，就是让距离最近的服务器响应请求，加快网络内容的传输。一般用于加速静态资源，如网站上面上传的图片、媒体等。
 
通常我们使用的是 [JsDelivr](https://www.jsdelivr.com/ "JsDelivr")，一个免费、可靠的 CDN 加速。下面是官网，主页面直接放了大量的使用方法：

![JsDelivr 官网主页](http://101.200.84.36/images/2022/05/23/202205231714065.png "JsDelivr 官网主页")
 
# 测试

首先找一张 GitHub 上的图片，我这里选了一张之前用过的 Zettlr 官网的图片，地址是

```markdown
https://raw.githubusercontent.com/chunshuyumao/202203bf/master/202203132034512.png
或者
https://github.com/chunshuyumao/202203bf/raw/master/202203132034512.png
```

效果如下——一般是看不到下面的图片的，也不排除图片可以显示，只是加载比较慢。

![未加速前 Zettlr 官网图片](https://raw.githubusercontent.com/chunshuyumao/202203bf/master/202203132034512.png "未加速前 Zettlr 官网图片")

要使用 CDN 加速，需要把地址改成下面的形式

```markdown
https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132034512.png
```

![JsDelivr 加速后的 Zettlr 编辑器官网主页图片](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132034512.png "JsDelivr 加速后的 Zettlr 编辑器官网主页图片")
 
可以看到，要加速 GitHub 的图片，加速地址是这样的

```markdown
https://cdn.jsdelivr.net/gh/github帐号名/仓库名@分支名/图片名
```

其中，分支名一般是主分支，也就是 master 或者 main，至于到底是哪个，要是看你的仓库的主分支。

# PicGo 加速

如果使用 GitHub 作为图床，建议配合 PicGo 使用。使用 PicGo ，需要修改 GitHub 的配置。打开 PicGo，转到 GitHub 设置，按照下面进行配置。

首先，分支名要写你的 GitHub 分支，我这里是 main 分支。其次写自定义域名，格式和上面说的一样。这样，PicGo 上传后会自动返回加速后的地址。

```markdown
https://cdn.jsdelivr.net/gh/GitHub帐号/仓库名@分支名
```

![PicGo 中的 GitHub 设置](http://101.200.84.36/images/2022/05/23/202205231727605.png "PicGo 中的 GitHub 设置")

# Hexo 加速

`cdn.jsdelivr.net` 加速也不一定完全可靠，比如前几天这个加速也样被污染了。`cdn.jsdelivr.net` 使用不了，我们可以改成 `fastly.jsdelivr.net` 或者 `gcore.jsdelivr.net`。如果使用 Hexo 管理静态博客，我们可能需要修改博客中的图片地址。

如果不嫌麻烦，可以使用文本编辑器直接全局修改。例如在 Linux 系统上使用 <ruby>SED（文件流编辑器）<rt>Stream Editor</rt></ruby> 全局修改：首先转到博客源文件（Markdown 文件）目录，然后输入修改代码。

```bash
$ cd Docment/NotePages/source/_posts # 进入源文件目录
$ ls | xargs -i sed -i 's/cdn.jsdelivr.net/fastly.jsdelivr.net/g' {} # 修改文件内地址
```

![SED 批量修改地址](http://101.200.84.36/images/2022/05/23/202205231740399.png "SED 批量修改地址")

如果是其他系统的用户，可以使用文件编辑器打开文件，然后使用全局修改。

不过这只是权宜之计，我希望还是使用 `cdn.jsdelivr.net` 只是在它使用不了的时候再改成其他加速。既然如此，我们就需要借助其他工具了。

[Hexo Fluid 主题](https://hexo.fluid-dev.com/ "Hexo Fluid 主题") 有 [注入功能](https://hexo.fluid-dev.com/posts/hexo-injector/ "Fluid 主题注入功能")，也就是下面用到的。如果想要真正了解该主题的注入功能，最好到 [官网](https://hexo.fluid-dev.com/posts/hexo-injector/ "Fluid 主题注入功能") 查看。

然后上 GitHub 用户 [PipecraftNet](https://github.com/PipecraftNet/jsdelivr-auto-fallback "PipecraftNet") 的仓库下载 `index.min.js` 或者 `index.js` 文件，或者直接复制里边的内容。这位大佬写了一个自动检测加速有没有用的脚本，会给自己的博客自动替换更好的 CDN 加速。

转到博客源文件根目录（也就是 source 目录），新建一个 js 文件夹，把下载的 `index.js` 文件放到里边，最好重命名。如果是直接复制内容，那就在 js 文件夹中新建一个扩展名为 `js` 的文件，把复制的内容粘贴到里边。

下面演示使用 axel 下载，并且重命名文件为 `change_cdn.js`。

```bash
$ cd Documents/NotePages/source # 转到源文件根目录
$ tree -L 1 # 查看是否有 js 文件夹，没有就新建
.
├── about
├── _drafts
└── _posts

3 directories, 0 files
$ mkdir js # 新建 js 文件夹
$ tree -L 1
.
├── about
├── _drafts
├── js
└── _posts

4 directories, 0 files
$ cd js
$ axel https://github.91chi.fun/https://raw.githubusercontent.com/PipecraftNet/jsdelivr-auto-fallback/main/index.js -o change_cdn.js
```

下载完之后，转到 Hexo 博客根目录，在根目录下创建一个 `scripts` 文件夹，在里边创建一个名为 `insert.js` 的文件（文件名随便取）。将后面的代码复制到 `insert.js` 文件内。`src` 后面写的是刚刚下载的文件的文件名，我的是 `change_cdn.js`。

```bash
$ cd ~/Documents/NotesPage
$ mkdir scripts
$ vim scripts/insert.js

hexo.extend.injector.register('head_begin', '<script defer src="/js/change_cdn.js"></script>', 'default');
```

保存退出，完成脚本的注入了。

输入 `hexo clean && hexo g -d` 进行博客部署，脚本会自己判断哪一个 CDN 更好用，然后自动修改。当然，这个只对博客的展示产生影响，不会直接修改你的博客内容。

# 后语

好像 `cdn.jsdelivr.net` 加速又可以访问了，其实已经卡了好几天了。