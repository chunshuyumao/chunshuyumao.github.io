---
title: Zotero文献管理插件安装
hide: false
tags:
  - Zotero
  - 文献管理
categories:
  - 论文
math: true
mermaid: false
date: 2022-03-15 20:38:45
index_img:
excerpt: 安装几个有用的 Zotero 插件
---

# 1. 前言

前面介绍了 Zotero[^1] 等文献管理工具的安装和简单使用。今天讲讲 Zotero 插件的使用。早前我有说过，Zotero 具有丰富的插件支持。由于当时为了介绍其他的两款文献管理工具，避免篇幅过长，所以没有介绍 Zotero 的插件。

# 2. 安装插件

在介绍安装插件之前，我们要知道，Zotero 插件通常都可以在官网[^2]找到。好的我们继续。

首先看没有安装任何插件的 Zotero 中文界面:

![原始中文界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152053611.png)

推荐安装浏览器插件 Zotero Connector :

![安装 Connector 插件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152055785.png)

![浏览器中的 Connector](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152056245.png)

我使用火狐浏览器，所以插件推荐的是火狐。安装 Connector 插件为的是后面更好地下载论文。安装好之后，你的浏览器右上角应该出现这个标志。

![Connector](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152104923.png)

现在打开任何一个可以查论文的东西——实际上，Zotero 不仅仅可以管理论文，还包括所有浏览器能嗅探的东西，例如网页——这里随便查找文献。右上角的图标变成了学者的帽子，点击之后会自动抓取。

![自动识别论文](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152107569.png)

![自动抓取](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152109547.png)

为了后面的示范。我还抓取了几篇论文。

![抓取的论文](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152111974.png)

Zotero 抓到是抓到了，但是 PDF 文件似乎没有进行重命名。我们希望它可以对我们的论文 PDF 进行重命名。这里我们需要下载一个插件，叫做 Zotfile[^3]。

![Zotfile 官网网址](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152114431.png)

下载完之后，打开 Zotero 的 `工具-->插件`，选择 `Install Add-on From File`，然后进行安装，每次安装之后都需要重启。

![选择 install from file](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152116952.png)

![安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152117551.png)

重新打开之后，选择要修改的文件，`右键-->根据父级元素重命名`。

![重命名](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152120959.png)

![重命名](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203160747860.png)

修改完之后，我们看到，标签里的作者名字是分开的，我们不希望姓名分开，所以安装下一个插件：<ruby>茉莉花<rt>Jasminum</rt></ruby>[^4]。这款插件不在官网，却不可或缺。到 GitHub 下载 `xpi` 文件。

![下载 xpi 文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152128362.png)

下载之后安装重启。右键小工具，可以修改。

![右键小工具](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152129563.png)

![名字已经正常了](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152130199.png)

还可以在 `首选项` 中修改配置

![茉莉花配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152132410.png)

最后下载 Better BibTex[^5]。如其名，这个和 $L^AT_ET$有关。不过不用担心，先安装再说。重启配置，选择 `BBT default citekey format`，允许拖拽引用，最后一直 `next` 就行了。安装完之后，就会看到 `CiteKey`，这是你引用的时候查找的关键字。当然也可以更改。

![Better BibTex](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152134383.png)

![重启配置，选择 BBT default citekey](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152144263.png)

![允许拖拽引用](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152145391.png)

![安装完成](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152234217.png)

到 `首选项` 中选择 `重命名链接的文件`。

![配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152148519.png)

到这里，我们的插件基本安装完了，如果你还有什么想要安装的，可以到官网查找插件或者 GitHub 上面查找。

对我而言，Zotero 还可以用来做文献阅读笔记，所以习惯性添加 MarkdownHere[^6] 插件。如果上不去，可以用 Gitee[^7]。同理，上面的插件如果上不了 GitHub ，可以搜索 `插件名 Gitee`，一般都有镜像。进入下载的文件夹或者解压的文件夹中的 src ，压缩 `common`、`firefox`、`chrome.manifestr`、`install.rdf`到随便一个文件夹，建议仍然使用 `MarkdownHere`，修改后缀为 `xpi`，然后进行安装。

![压缩](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152221183.png)

![MarkdownHere 插件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152223699.png)

安装完之后可以使用 Markdown 做笔记，做完笔记之后全选使用 <kbd>CTRL</kbd>+<kbd>ALT</kbd>+<kbd>M</kbd>进行格式转化。选择一篇论文，单击下边的笔记 ( Notes ) --> 添加 ( add )，做笔记。

![全选原始格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152232204.png)

![MarkdownHere渲染的格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152232962.png)

哈哈，果然简单是吧？其实蛮喜欢 Markdown Here 的，我的邮箱客户端就安装了它。

如果想更改你的导出格式，可以到 `首选项--->引用--->更多样式` 查找你的参考文献格式。默认国内使用 GB2005 。

![添加格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152241815.png)

![国标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203152242873.png)

Zotero 的引用格式会安装到自己目录的 `Styles` 文件夹，所以别忘了你的 Zotero 安装到哪里——后面我们会用到。

# 3. 后语

Zotero 对与一个经常看文献的人来说确实是一个好东西。我现在还是要感慨，为什么 WPS 没有相应的 Zotero 插件？这是逼我使用 Markdown 写论文呀——所以下一篇就打算写写如何用 Markdown 写论文。

# 相关网址

[^1]: https://www.zotero.org/
[^2]: https://www.zotero.org/support/plugins/
[^3]: http://zotfile.com/
[^4]: https://github.com/l0o0/jasminum/releases
[^5]: https://retorque.re/zotero-better-bibtex/
[^6]: https://github.com/adam-p/markdown-here
[^7]: https://gitee.com/mirrors/markdown-here
