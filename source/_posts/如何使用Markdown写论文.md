---
title: 如何使用Markdown写论文
hide: false
tags:
  - Zettlr
  - Markdown
categories:
  - 论文
math: true
mermaid: false
date: 2022-03-15 21:39:18
excerpt: Markdown 妙用
---

# 1. 前言

Markdown 作为新兴的写作工具越来越受人的青睐，用 Markdown 写成的纯文本文件可以快速移植，省去再排版的烦恼。既然如此，能否使用 Markdown 进行论文写作呢？答案是肯定的。Markdown 可以直接使用 $L^AT_EX$ 的数学公式排版，同时还可以搭配 Mermaid 进行画图。使用 Markdown 写完自己的文章之后，可以通过 Pandoc 进行格式转换，轻松导出成各种格式。Markdown 还可以配合 R 语言进行专业绘图和排版。

本次我们将借助 Zettlr 编辑器完成论文的基本排版和写作。如果你会 $L^AT_EX$ 的话请往他处。

# 2. 工具准备

要准备的工具主要有以下几种：

1. Zettlr Markdown 编辑器[^1]。
2. pandoc-crossref 交叉引用软件[^2]。
3. Zotero 文献管理工具[^3]。

下面我们会逐一介绍这些工具的使用。

 2.1. Zettlr Markdown 编辑器

Zettlr 是一款专门面向写作者和研究者的 Markdown 编辑器，由德国学者 Hendrik Erz 创建。理由很简单：他对现在的文字处理软件感到不满，他想要一个可以让他“专注于写作和阅读”的编辑器。秉承着自己动手丰衣足食的理念，这位大佬开始了自己的编程之路，于是 Zettlr 这个“二十一世纪的 Markdown 编辑器”诞生了。由于是为了学者和写作者而作，Zettlr 自带各种强大的属性。这里就不探讨它的强大功能了，有需求请去官网[^1]。

现在的 Markdown 编辑器数不胜数，为什么要选择 Zettlr ？其实理由很简单：它支持文献引用。写论文的时候，我们需要对一些参考文献进行引用。大多数时候靠 Word 可以快速使用各类文献管理器的引用插件，然而我更偏向 WPS。去年把自己的文献管理器从 NoteExpress 迁到 Zotero，谁知道 Zotero 没有 WPS 的插件。看了看相关开发者的反馈，据说他们找金山公司，希望开发 WPS 的 Zotero 插件，结果被告知 WPS 的 <ruby>应用程序开发接口<rt>Application Programming Interface</rt></ruby>( API ) 和 Office 的是一样的。也就是说，金山公司认为如果 Zotero 的插件可以在 Office 中使用，自然也可以在 WPS 中使用。可惜事实并非如此。前一阵我发现国外的大佬们在探讨这件事的时候说，原来 WPS 有开发者文档，可只有中文没有英文，他们看不懂。太可惜了，我看得懂。

那一阵子还在使用 Windows，写论文还可以用 WPS 配合 NoteExpress。现在使用了 Linux 系统，只能硬着头皮使用 Markdown。

说了那么多废话，其实就是想说，接下来论文就在 Zettlr 编辑器上完成了。

![Zettlr 官网](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132034512.png)

![Zettlr 编辑器界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132036443.png)

Zettlr 是 <ruby>WYSIWYM<rt>What You See Is What You Mean</rt></ruby>(所见即所想) 的 Markdown 即时渲染编辑器，有别于 Office Word 和 WPS Word 的 <ruby>WYSIWYG<rt>What You See Is What You Get</rt></ruby>(所见即所得)。唯一的不同是，它保留了 Markdown 的原始格式。

 2.2. YAML Front-matter

我们可以看到 Zettlr 文档的头部有一部分由三条横杠分隔，它叫做 <ruby>YAML<rt>Yet Another Markup Language</rt></ruby>Front-matter，中文名不知道，暂且叫做“标头”。这一部分使用的是 YAML 语法，可以配置文档信息。这一部分可以在官网教程查看。简单说一些配置：

```yaml
---
title: 标题
author: 
	- 作者1
	- 作者2
keywords:
	- 关键字2
	- 关键字2
abstract: |
	这里写摘要
---
```
注意，YAML 语法的特点是，冒号之后的文字与冒号有一个空格，还有缩进：

```yaml
# 正确操作
key: value
keys:
	- value1
	- value2
	- value3
	- value4

# 错误示范
key:value
keys:
-value1
-value2
	-value3
  -value4
```

后面与 YAML 有关的操作，如果你发现自己的配置不起作用，请检查是不是少了一个空格，或者缩进不对。

 2.3. 引用文献

要引用文献，我们需要打开 Zotero 文献管理器，选择 Better CSL JSON 或者 Better BibTex 格式导出文献库：

![导出文献库](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132055432.png)

![选择 CSL JSON 或者 BibTex 格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132056233.png)

记住勾选 Keep Updated 保持更新，保存到你喜欢的文件夹。

回到 Zettlr，选择 文件---><ruby>首选项(偏好)<rt>Preferences</rt></ruby>，或者直接快捷键 <kbd>CTRL</kbd>+<kbd>,</kbd> 打开首选项，选择 <ruby>引用<rt>Citation</rt></ruby> 模块，在 <ruby>引用库<rt>Citation Database</rt></ruby> 下选择你刚刚导出的库，下一行填写你的参考文献格式的路径。

![首选项](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132101687.png)

![填写引用文献的路径和类型](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132102458.png)

参考文献格式是我们论文最后写的文献的格式，国内通常使用 GB2005 。可以在 Zotero 中下载，然后在 Zotero 安装路径里边找 Styles 文件夹，找到你想使用的 <ruby>引用格式(CSL)<rt>Citation Style Language</rt></ruby> ，复制路径到上面需要填写的地方。Zotero 的使用可以参考我之前写的文章。

> Zotero 使用的文章
>
> [Zotero文献管理插件安装](https://chunshuyumao.github.io/2022/03/15/Zotero%E6%96%87%E7%8C%AE%E7%AE%A1%E7%90%86%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85/)
>
> 还有一篇安装的文章，原始 Markdown 文件不见了，懒得再写，要看只能去公众号：椿树与猫 查看。

到这里，我们的准备就完成了。Zettlr 中引用文献使用 `[@CiteKey]`，其中 `CiteKey` 就是我们之前在 Zotero 安装的 Better BibTex 提供的独一无二的标识，包含引用的文献的一些信息，例如文献标题或者作者、年限什么的。看下面，我只是输入了 `@` 它就自动弹出了文献库的文献，这时候你只需要再添加一些信息就可以快速查找自己想引用的文献。

![尝试引用](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132110871.png)

引用完之后， Zettlr 的右边会出现一个引用列表，如果没有，也可以点它出现，就像这样：

![引用列表](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132112145.png)

如果想要修改自己的引用格式，就更改 CSL-Style 的路径。

这是最基础的使用，我们可以使用 <kbd>CTRL</kbd>+<kbd>E</kbd> 导出试试(也可以在 <ruby>文件<rt>File</rt></ruby> 中选择导出)：

![导出](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132114242.png)

![导出](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132117406.png)

导出之后，它自己给我们标号和更改参考文献格式。

![PDF 格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132119048.png)

 2.4. 修改 Word 模板

有些时候，我们需要各种 Word 格式，可以通过修改 Pandoc[^4] 默认模板完成。首先你的电脑要安装 Pandoc ，并配置好环境变量。<kbd>Windows</kbd>+<kbd>R</kbd>输入 CMD ( 不可使用 Powershell ) 或者在 Linux 打开终端，输入一下代码

```shell
pandoc --print-default-data-file=reference.docx > ref.docx
```

> Zettlr 内置 Pandoc，但是我们要定义自己的 Word 模板需要下载 Pandoc。如果不想安装，可以从 Zettlr 安装目录找找有没有 Pandoc 可执行文件。我不知道 Windows 系统可不可以找到，无论如何，还是建议直接直接安装 Pandoc ——大不了用完再卸载。

正常情况下，你所在的目录会出现一个 ref.docx 的文件，那就是你的 Word 模板文件。使用 WPS 或者 Office Word 打开，然后修改格式，不是在正文修改，而是类似下面这个模板修改

![修改格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203132129394.png)

修改的具体格式得看你自己了，因为没有万能的论文格式。

一般而言，三线图是不可以直接修改的：如果真的使用 Markdown 写论文，导出成 Word 格式之后还需要手动修改三线图。

修改完之后，打开 Zettlr 的<ruby>资源管理器<rt>Assets Manager</rt></ruby>，选择<ruby>导出<rt>Export</rt></ruby>中的 Word 设置，在末尾添加

```yaml
reference-doc: /home/chunshuyumao/Zotero/forzettlr/ref.docx
```

`reference-doc`后添加的是你的 ref.docx 所在路径，上面写的是我的路径，别抄。

默认情况下导出的参考文献的标题是英文 Reference。如果你想改成中文“参考文献”，可以在标头或者 Word 导出设置中添加：

```yaml
reference-section-title: "参考文献"
```

导出设置中的 `shift-heading-level-by` 需要注意一下：它的意思是修改标头的等级。一般来说，一个`#`表示一级标题，两个表示二级标题。如果你想让一个 `#` 表示三级标题，那就改成 `shift-heading-level-by: -3`；如果想让 `#` 表示一级标题，那就改成 `shift-heading-level-by: 3`。大家可能会奇怪：为什么会有这种神奇的语法？我想用一级标题就用一级标题，想用二级就用二级，为什么还搞得这么花里胡哨？其实不然，我们写 Word 的时候，通常是从一级标题开始，但是在网页上，一级标题太大了，网页上的最高一级的标题实际上是 Word 上的三级标题。所以在写的时候，我们可以使用三级标题，导出为 Word 的时候直接修改<ruby>偏移量<rt>shift-heading-level-by</rt></ruby>就免去了大量修改的麻烦。

 2.5. 交叉引用

我相信，本科生基本没有考虑过什么是交叉引用。很简单，即使是经常写论文的我们专业也很少会碰见大量的图片修改，或者论文存在大量图片。大多数情况下，我们写论文都是手动写索引，例如"图1""图2""图3""表1""表2""表3"。

![手动写图片索引](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203160936531.png)

然而这种手动操作，存在一个很大的弊端：我不想手操。

好的，懒人兴致勃发，接下来就是如何偷懒：让 Markdown 自己给我们标号。先看看原始状态：

```Markdown
![我大南开](file:///home/chunshuyumao/Pictures/signture.png)
```

![效果如下](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203160944232.png)

![导出为 Word 之后](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203160945421.png)

我们想让它自动标号，需要下载一个交叉引用的插件，叫做 pandoc-crossref [^5]，老规矩——上不去就去<ruby>码云<rt>Gitee</rt></ruby>找镜像。下载之后把它放到一个安全的地方，以免以后被你误删——建议和 Pandoc 可执行文件放一起，这样好找。

在自己的文件标头写

```Markdown
filters:
	- /usr/bin/pandoc-crossref
```

其中 `/usr/bin/pandoc-crossref` 是你的 pandoc-crossref 路径 。

> 注意：Windows 系统使用的分隔符是 \ ，我现在在 Linux 系统，所以使用 / 。希望使用时记得区别。

然后在自己的图片以后这样写

```Markdown
![我大南开](file:///home/chunshuyumao/Pictures/signture.png){#fig:id}
```

其中 `id` 是一个**没有空格**的标识，我使用 `this` 作为标识：

```Markdown
![我大南开](file:///home/chunshuyumao/Pictures/signture.png){#fig:this}
```

> 注意，`{#fig:id}`没有任何空格，千万千万不要随意加空格！马克思保佑你！

在文中引用图片使用 `[@fig:id]`，格式和文献引用一样：

![引用图片](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161000025.png)

![导出 Word 效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161001519.png)

文中的图片交叉引用好是好了，但是我希望它显示的是 `Figure.1` 而不是 `fig.1` 呢？简单，在标头写下

```yaml
figPrefix: Figure
```

![修改前缀](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161009283.png)

那如果我想图片标注的是"图'"而不是"Figure"呢？Easy，修改

```Markdown
figureTitle: "图"
```

![修改](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161009608.png)

![效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161010961.png)

表格我们也依样画葫芦：

```yaml
tableTitle: "表"
tblPrefix: "表"
```

但有点不同：表格的注释是写在表的下面，以冒号( : )开始，加一个空格，然后是注释，再加一个空格，然后才是 `{#tbl:id}`，引用使用 `[@tbl:id]`。

![Markdown 格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161014636.png)

![效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161014297.png)

好是好了，但是每次用 Markdown 写论文，我难道都要写这么多东西？非也，写一个 YAML 文件，把这些信息写进去

```yaml
figureTitle: "图"
figPrefix: "图"

tableTitle: "表"
tblPrefix: "表"

filter: 
	- /usr/bin/pandoc-crossref
reference-section-title: "参考文献"
```

![新建一个 YAML 文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161020560.png)

保存之后打开<ruby>资源管理器<rt>Assets Manager</rt></ruby>，修改<ruby>导出<rt>Export</rt></ruby>中 Word 配置的 meta-files (元文件)，写下你的 YAML 文件路径，我的是 `/home/chunshuyumao/Zotero/forzettlr/basic.yaml`.

```yaml
metadata-files:
- /home/chunshuyumao/Zotero/forzettlr/basic.yaml
```

![添加元文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161023406.png)

其实标头的东西可以都写到 YAML 文件中——YAML 中的文件其实也可以直接写到导出设置中，单独写出是方便而已。

别人的图片引用都有"图 1.1"，我也想这么玩怎么办？简单，在 YAML 文件中添加

```Markdown
chapters: true
```

即可

![分节](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161033526.png)

我不想写标题，如何让 Markdown 自己生成标题？

在<ruby>资源管理器<rt>Assets Manager</rt></ruby>或者 YAML 文件中写入

```yaml
number-sections: true
```

![number-sections](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161036663.png)

![自动标号](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203161037353.png)

好的，我不会再满足你任何要求了，有需求请去 Zettlr 官网[^1]和 Pandoc 官网[^4]查看教程，再见！

# 3. 后语

例行公事，Markdown 作为一个简单的排版工具确实不错，现在还是六大论文格式之一。如果不是非常专业的话，也许试试 Markdown 比 $L^AT_EX$ 更轻松？如果你是 $L^AT_EX$ 大佬，请随意～

# 相关网址

[^1]: https://www.zettlr.com/
[^2]: https://github.com/lierdakil/pandoc-crossref/release/
[^3]: https://www.zotero.org/
[^4]: https://pandoc.org/
[^5]: https://github.com/lierdakil/pandoc-crossref/releases/
