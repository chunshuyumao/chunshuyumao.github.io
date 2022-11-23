---
title: 'Pandoc: 从 Markdown 到其他格式'
hide: false
tags:
  - Pandoc
  - Markdown
categories:
  - Markdown
math: false
mermaid: false
date: 2022-11-20 09:15:43
excerpt: 使用 Pandoc 将 Markdown 转换到 DOCX 和 HTML 格式
---

# 前言

[Markdown](https://markdown.com.cn/) 是一种轻量级的标记语言，可以在纯文本文档中添加格式化元素。Markdown 最大的特点就是专注于文字，而且因为纯文本格式，可以纳入版本管理。日常使用的 DOCX 格式是二进制格式，没法进行版本管理。纯文本意味着只要是一个编辑器便可以打开并对 Markdown 进行编辑和查看。

> 
> 二进制无法进行版本管理指的是，无法查看版本之间的区别，并非直接加入版本管理。
>

写作的时候，可以先使用 Markdown 进行写作，然后转换成其他需要的格式，实现一个文件满足多种格式，避免一点点修改原格式。转换格式必不可少 [Pandoc](https://Pandoc.org) ，一款支持多格式转换的工具。现在很多的 Markdown 编辑器基本依赖的就是 Pandoc。了解如何使用 Pandoc 可以控制自己的 Markdown 输出格式，更随心所欲。之前自己使用 [Zettlr](https://Zettlr.com)，Zettlr 帮解决了大部分的 Pandoc 问题，所以没有太在意 Pandoc。现在从 Zettlr 脱离出来，开始探究 Pandoc 的使用。由此，今天就是要对 Pandoc 做一个介绍，主要针对的是如何配合 Markdown 转化成 DOCX 格式和 HTML 格式。前者一般是学生必须的格式，所以必须得介绍；后者是用 Markdown 写作时的日常预览。


# Pandoc 安装

[Pandoc](https://Pandoc.org) 被称作文件格式转换的瑞士军刀，非常锐利简便，如果对细节要求不高，一般来说一行命令就可以完成许多格式之间的转换。介绍过 Markdown 的，基本都会说到 Pandoc，似乎这两者便是天生的一对。

Pandoc 是一个命令行工具，所以下载、安装之后是无法打开图形界面的，甚至看不到命令行界面。[官网](https://pandoc.org/installing.html) 有介绍如何进行安装，不过我更喜欢在 [GitHub](https://github.com/jgm/pandoc/releases/latest) 上直接下载。

![GitHub 上最新的 Pandoc 对应各个系统的软件包](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120095003.png)

安装完之后需要进行环境变量的设置，方便在命令行打开。如果使用官网推荐的方式进行安装，即使用包管理器安装，一般在命令行上可以直接运行，只需要打开命令行即可。在 Linux 和 MacOS 上可以找一个叫做终端(Terminal)的软件，点击即可进入命令行模式；在 Windows 系统上则可选择 CMD 或者 Powershell。Pandoc 导出 DOCX 文件格式使用 Powershell 有毛病，会出现乱码，所以推荐使用 CMD。因此 Windows 系统可以打开搜索找到叫做 `命令行` 的软件，打开即可。

打开命令行之后在命令行上输入 `pandoc --version` (Windows 系统使用的 Pandoc 命令是 `pandoc.exe --version`，后面所有指令在 Windows 上都写作 `pandoc.exe` 而非 `pandoc`)出现类似以下的界面则表示没问题：

![Pandoc 安装成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120100023.png)

## 环境变量配置

如果得到的提示是 `某某命令找不到(Not Found)` 或者 `未知命令` 这类错误，那说明环境没有配好，比如手动到 GitHub 下载。如果是手动下载，下载后的包需要解压缩(zip, tar.gz)，解压之后放到指定的地方，然后进行环境变量的配置。

### Linux 配置

在 Linux 上可以放到 `~/.local/bin/` 文件夹里面。首先打开命令行，输入 `ls ~/.local/bin` 查看路径是否存在。如果提示:

>
> 命令行中需要输入的命令会使用 `$` 标注起来，没有其他说明默认非 `$` 开头的都是命令的输出结果。
>

```bash
$ ls ~/.local/bin
ls: cannot access '/home/chunshuyumao/.local/bin': No such file or directory
```

类似的错误提示，说明你的路径下没有这些文件夹。这时候可以输入 `mkdir -p ~/.local/bin`，直接创建多级文件夹。找到自己下载的 Pandoc 的压缩包，一般是 `tar.gz` 结尾。然后 `ls` 确实有这个文件，输入 `tar` 指令进行解压。如下:

```bash
$ ls
Desktop    Downloads   Music          Pictures  Templates  Zotero
Documents  miniconda3  pandoc.tar.gz  Public    Videos
$ tar -zxvf pandoc.tar.gz -C ~/.local/bin
pandoc
```

其中 `-C` 表示解压的位置。在自己的命令行配置文件 `.bashrc` 中写下 `PATH=$HOME/.local/bin:$PATH` ，关闭命令行，重新打开试试 `pandoc --version`。

>
> 如果使用的其他的 Shell，例如 Zsh、Ksh 等，就需要在特定的配置文件 `.zshrc` 和 `.kshrc` 进行配置。
>

如果进行这些配置之后还是出现错误，那请重头开始。

### Windows 配置

如果是 Windows，直接解压放到任何一个目录，比如放到 `D:\Applications\Pandoc` 文件夹。打开 `系统属性` 界面，可以使用 Windows 自带的搜索直接搜 `系统属性` 几个大字。打开之后选择 `高级->环境变量->系统变量->Path(或者PATH)` ，双击 `Path` 选择新建，输入自己的 Pandoc 放到的位置，选择确定即可。

# Pandoc 使用

安装好 Pandoc 之后就可以进行简单的使用了，接下来我会使用 Linux 进行演示，但是所有操作在 Windows 的 CMD 上一样适用。

首先创建一个 Markdown 文件作为演示，文件拓展名是 `.md`。以下是用于演示的 Markdown 文件的内容：

```markdown
# Markdown 语法

*拉丁文斜体* 正字

**着重加粗** 无所谓

***拉丁文斜体加粗*** 无所谓的正字

>
> 子曰：我没说过
>

[Markdown](https://markdown.com.cm) 网址

![高清大图](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120110320.png)

|姓名|学号|年龄|
|:----:|:----:|:----:|
|张三|001|18|
|李四|003|17|
|王五|004|18|
: 花名单

## 二级标题

```

打开命令行，确认路径下有我们的 Markdown 文件。然后进行转化。转化时输出文件用 `-o outout.format` 表示，`-o` 是 `--output` 的缩写。

```bash
$ ls
test.md
$ pandoc test.md -o test.html
$ pandoc test.md -o test.docx
$ ls
test.md test.html test.docx
```

> 完整写法应该是 `pandoc --from markdown --to html test.md -o test.html`. 如果不指定类型，Pandoc 会通过扩展名自动判断。转换到 DOCX的完整写法是 `pandoc --from markdown --to docx test.md -o test.html`。也可以使用更短的命令 `pandoc -f markdown -t html test.md -o test.html`

![动图演示](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120111452.gif)

使用 WPS 或者 Office 打开 DOCX 文件查看，效果好像还行。

![Markdown 转换成的 DOCX 文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120111820.png)

同理使用 浏览器打开 HTML 文件。平心而论，很丑，丑的不行，根本看不下去。

![Markdown 转换成的 HTML 文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120112104.png)

这不全怪 Pandoc，是我们的操作有问题。因为有些格式不止由一个 Markdown 文件生成，而是多个文件组合。如果转换成 HTML 只使用一个文件，我们需要告诉 Pandoc 我们只需要一个文件，所以转换成 HTML 的正确操作应该是：

```bash
$ pandoc -s test.md -o test.html
```

命令中的 `-s` 是 `--standalone` 的缩写，表示单文件模式，不需要多个源文件组合。再看看转换之后的文件：

![使用单文件模式转换成 HTML 格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120112602.png)

这次的转换就很好看了。我开了暗夜模式所以网页看起来是黑的，其实是白的。

![正常情况下的预览](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120112732.png)

转换成 DOCX 的时候也需要单文件模式，但是 Pandoc 默认加上了。默认的 HTML 模板其实蛮好看，没啥需要改的。仔细看，转换单文件模式的 HTML 时，命令行还出现了这样的提示：

```bash
$ pandoc -s test.md -o test.html
[WARNING] This document format requires a nonempty <title> element.
  Defaulting to 'test' as the title.
    To specify a title, use 'title' in metadata or --metadata title="...".
```

这提示我们可以给文件加一个标题，语法是 `--metadata title="标题"`。可以用 `-M` 代替冗长的 `--metadata` 命令，在命令行加。我们试试：

```bash
$ pandoc -s -M title="Convert from Markdown to HTML" test.md -o test.html
```

![添加标题](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120113424.png)

可以看到，渲染的网页给我们添加了标题:

![网页添加标题](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120113457.png)

同理转换成 DOCX 看看：

![DOCX 添加标题](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120113702.png)

只是使用基本的语法到这里就差不多了，要使用其他功能就需要继续往下看。

# Pandoc 进阶

进阶部分我们会介绍如何使用 Pandoc 进行文献引用、链接，自动给标题标号、分章节，修改链接格式，甚至图片、表格、公式交叉引用、导出格式等等，也会使用到其他的工具，到时会细说。

## 文献引用

这里的引用(citation)指的是写论文时的参考文献，与 `>` 代表的直接引用名人名言不同。可以在官网上看到有关引用的介绍。有什么问题最好也是官网查看，到处在网上搜属实是下下策。

![官网上对引用的介绍](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120124611.png)

从官网上了解到，想要引用需要：

1. 包含引用的文件，比如我们的论文 Markdown 格式。
2. 参考文献(bibliography)源文件，可以是参考文献文件，也可以使用 `-M` 标注。用 `-M` 标注太低级了，后面的演示会直接使用 [Zotero](https://www.Zotero.org) 导出的文献库。
3. CSL 引用格式文件。这是可选的。
4. 使用 `--citeproc` 进行参考文献的引用处理。

参考文献格式有多个类别，可查看下表。Pandoc 处理参考文献时会把其他格式转换成 JSON ，所以为了简便可以直接使用 JSON 格式，少了中间商。那就准备文献库吧。

|格式|扩展名|
|:----:|:----:|
|BibLaTex|.bib|
|BibTex|.bibtex|
|CSL JAON|.json|
|CSL YAML|.yaml|
|RIS|.ris|
: 参考文献格式

### 从 Zotero 中导出文献库

导出文献库需要下载 Zotero 作为文献管理器，然后需要的文献放到 Zotero 中。Zotero 是一个大命题，这里不会过多介绍，如有兴趣可以看看之前写的文章 {% post_link Zotero文献管理插件安装 %}。安装 [Better BibTex for Zotero](https://retorque.re/zotero-better-bibtex/) 插件方便使用 `citationkey`。`citationkey` 是 Pandoc 实现引用的关键东东，就像 DOI(digital object unique identifier，数字对象唯一标识符) 一样，每一篇文献都有一个对应的 `citationkey`，独一无二。

![citationkey](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120130813.png)

安装好插件之后就要导出文献库。Zotero 里打开 `文件(File) -> 导出库(Export Library)`，选择 `Better CSL JSON` 格式，然后勾选 `保持更新(Keep updated)`，`OK` 即可。如果没有这个格式，建议看看是不是没有安装 Better BibTex 这个插件。保存到一个指定目录，我保存到 `/home/chunshuyumao/Documents/Pandoc/ZoteroLibrary.json`，后面要用到。

![选择导出文献库](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211161958383.png)

![选择格式，保持更新](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211162000373.png)

![保存到某个目录](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211162011533.png)

文献库是一个 JSON 格式的文件，可以打开瞧瞧。下面展示的是文件中的信息，截了一篇文献的信息。

![一篇文献的引用信息](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211162045423.png)

Pandoc 识别引用的语法是 `[@citationkey]`。意思是在 Markdown 文件中引用一篇文献需要这样写：

```markdown
这是文章内容，巴拉巴拉巴拉。研究[@AlfredRodrigo-1702]认为巴拉巴拉巴拉。
```

Pandoc 渲染之后自动标号，而且在后面生成了参考文献。

![Pandoc 渲染后](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211162058293.png)

需要注意的是，如果没有给 Pandoc 指定文献格式(Citation Style language，CSL) ，Pandoc 默认使用 Chicago Manual of Style。格式可以到 Zotero 中下载。一般来说，国内使用的是国标(GB)，至于哪个版本就看需求了，我一般用 2005 年的 numeric. 下载之后的 CSL 文件在 Zotero 安装位置下的 styles 文件夹。

![下载格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211162108573.png)

![CSL 位置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202211162111253.png)

复制想要用的 CSL 文件到某个位置，这里复制到和文献库同一位置，即 `/home/chunshuyumao/Documents/Pandoc`。

### 使用引用

准备好文献库，在我们的 Markdown 文件中添加以下行：

```markdown
## 二级标题

这里即将引用一篇文献[@DefiningCellTypes]。是的没错。

```

![全文如上](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120133037.png)

Pandoc 将 Markdown 转换到 HTML 并使用引用，除了文献库和 CSL 文件，还需要加上 `--citeproc` 进行处理，所以语法如下：

```bash
$ pandoc -s -M title="Convert from Markdown to HTML" --citeproc --bibliography /path/to/lib.json --csl /path/to/style.csl test.md -o test.html
```

上面的命令行太长，我们可以使用换行，让它看起来更正常一点：

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

这样看起来舒服多了。同理，把 `test.html` 换成 `test.docx` 就可以生成 DOCX 格式了。

![命令行转换](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120133530.png)

![HTML 格式效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120133641.png)

![DOCX 格式效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120133758.png)

初步完成了引用，写论文一般要求引用可以点击跳转。这个也可以做到，只需要添加 `-M link-citations=true`。

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

![动图演示点击跳转](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120134937.gif)

这里有个毛病，点击跳转的 DOCX 文件在 WPS Office 中没办法跳转，但在 Microsoft Office 中没问题。可能使用的 Linux 版 WPS 有问题，因为手机版也可以跳转。

## 自动给标题标号

处理完文献我们希望，导出的 HTML 和 DOCX 的标题自动给标题标号，这种东西没必要手动操作，直接让 Pandoc 处理。查看官网，想让 Pandoc 标号只需要添加一行命令 `-N` 或者 `--number-sections`。

![自动标号](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120135849.png)

好家伙，叠 Buff 开始：

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -N \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

![自动标号成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120140103.png)

后面我会用 HTML 演示，DOCX 和 HTML 差不多，有区别时我才会单独拎出 DOCX，否则操作共享。

## 交叉引用

写论文不免使用图片、公式和表格，特别是 `如图几` 这类的语句。如果原本你有三张图，并且已经标好号了，论文内有五个 `如图几`。不想你又在开头添加一张图片。这时候你需要更改所有的 `如图几`，因为在开头添加图片就把原来的标号顺序全都弄错了。为了避免这种情况，我们一般选择交叉引用。坏消息是 Markdown 和 Pandoc 都不支持交叉引用，好消息是有一个 Pandoc 插件可以支持。这个插件就是 [Pandoc-crossref](https://github.com/lierdakil/pandoc-crossref)。

### 下载 Pandoc-crossref

先到 [GitHub](https://github.com/lierdakil/pandoc-crossref/releases) 下载 Pandoc-crossref。注意下载时版本对应，看看自己的 Pandoc 是什么版本，使用 `pandoc --version` 查看。我的是 `2.19.2` 可以直接下载最新的 Pandoc-crossref。不同版本编译的 Pandoc-crossref 一般不能交叉使用。

![Pandoc-crossref 下载](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120142401.png)

下载完成后建议和 Pandoc 放一起方便管理。如果使用的是不同的目录，比如你不愿把它们放到一起，那请记住位置，并为 Pandoc-crossref 设置环境变量，否则下面使用 Pandoc-crossref 需要带上全路径。

> 全路径指的是，把文件的绝对路径写上。配置环境变量本质上就是免了写全路径。

### 使用 Pandoc-crossref

可以查看 Pandoc-crossref 的 [用法](https://lierdakil.github.io/pandoc-crossref/) 。这里我会说明基本用法。首先是最常用的图片引用，语法如下：

```markdown
![图片注释](图片路径){#fig:id}
```

正文中使用引用只需用 `[@fig:id]` 即可。格式和参考文献的引用一样，参考文献由 `--citeproc` 处理，为了避免 Pandoc 把交叉引用当作参考文献解析，我们需要在使用 `--citeproc` 之前使用 pandoc-crossref, 命令行语法为 `--filter pandoc-crossref`。`id` 是一个拉丁字母的组合，不可使用中文、数字和下标等等。如下为图示:

![图片引用](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120143501.png)

注意，一定要在使用 `--citeproc` 之前使用 pandoc-crossref。

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -N \
  --filter pandoc-crossref \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

![导出效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120143827.png)

Pandoc 渲染引用的图片标号用 `fig.` 代替，并在图片注释前添加 `Figure.`。中文写作想使用 `图` 而不是 `fig` 怎么办？添加 `-M figPrefix="图"`。

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -N \
  -M figPrefix="图" \
  --filter pandoc-crossref \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

![修改前缀](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120145040.png)

![修改引用的前缀](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120144908.png)

想把图注释中的 `Figure` 也改中文？野心不小嘛，添加 `-M figureTitle="图"`。同理，公式、表格的用法也一样，甚至还有章节、列表的引用，见下表：

<div>

![](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120145251.png)

![](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120145517.png)

Pandoc-crossref 支持的交叉引用(部分)
<div>

在 Markdown 文件中添加如下引用：

![使用交叉引用](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120153534.png)

命令行输入

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -M figPrefix="图" \
  -M figureTitle="图" \
  -M tblPrefix="表" \
  -M tableTitle="表" \
  -M eqnPrefix="公式" \
  -N \
  --filter pandoc-crossref \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

效果如 `交叉引用1` 和 `交叉引用2`。想必有人发现不同了，图 2 的表格和图片都标上了 `1.1` 表示第一章的第一张图，而图 1 没有。这是因为我加了分章节 `-M chapters=ture`，这样图片和公式可以按照章节(其实就是标题)来划分，且第一张图的数学公式没有渲染。这个不急，后面会说怎么渲染数学公式。

![交叉引用1](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120152620.png)

![交叉引用2](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120152413.png)

至此，交叉引用已经完成了，该解决数学公式了。

## 数学公式

Pandoc 渲染数学公式需要借助外力。Pandoc 渲染数学公式需要显式指定渲染引擎。如下

1. mathjax 
2. mathml
3. webtex
4. katex
5. gladtex

这几个有好有坏，看个人取舍，我一般使用 MathJax，这个引擎渲染的范围广一点，使用的也比较多。至于语法，添加 `--mathjax` 即可。还可以添加网址，例如 `--mathjax=URL`。为什么还要有网址？因为这东东是“外力”，Pandoc 不自带，它会调用内置的网址。由于这些网址都是国外的，所以很不幸可能会慢，大概率会慢，很慢。那怎么解决？其实只需下载一份 JavaScript 文件保存即可。当渲染的时候，提供路径到保存的引擎文件，没有借助网络进行渲染。

到 [SourceForge](https://sourceforge.net/projects/mathjax.mirror/) 下载 MathJax 压缩包，解压之后重命名为 `MathJax` 并移动到某一个文件夹，可以放到 `/home/chunshuyumao/Documents/Pandoc/` 。

![SourceForge](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120155048.png)

放好之后使用指定位置，现在我们的命令行如下：

```bash
$ pandoc -s \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -M figPrefix="图" \
  -M figureTitle="图" \
  -M tblPrefix="表" \
  -M tableTitle="表" \
  -M eqnPrefix="公式" \
  -N \
  --mathjax=/home/chunshuyumao/Documents/Pandoc/MathJax/es5/tex-mml-chtml.js \
  --filter pandoc-crossref \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.html
```

不下载其实也可以用，就是看个人网络问题。如果不下载，就只需要加 `--mathjax`。

![数学公式渲染](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120155705.png)

## 论文 DOCX 模板

刚开始时我们就发现生成的 DOCX 好像不咋地，且写论文一般有要求，例如字号、字体等，靠 Pandoc 默认的模板很难满足需求，我们需要自己的模板。Pandoc 考虑到了，提供 `reference-doc` 引入模板。

![reference-doc 引入 DOCX 模板](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120160438.png)

### 导出 Pandoc 默认模板

首先导出 Pandoc 模板，它预设了很多格式，直接修改即可。使用 Windows 系统需要注意，这一步的导出只能使用 CMD，不可使用 Powershell，后者会出现乱码。在命令行中输入：

```bash
$ pandoc --print-defatul-data-file ref.docx > ref.docx
```

用 WPS 或者 Office 打开，看看预设的模板如何：

![获取模板，并打开](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120160858.png)

![打开木板预览](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120161032.png)

没有表格对应的模板，所以三线表需要导出成 DOCX 后再手动进行修改。修改 DOCX 模板不在本篇文章的范围，所以大家最好自己去试。

修改之后的模板也放到 `/home/chunshuyumao/Documents/Pandoc/` 下边，待会再放个大招。

### 使用自己的模板

假设已修改完模板可以使用了。这个模板只有 DOCX 使用，所以导出成 HTML 就不必加入了。语法是 `--reference-doc=/path/to/template.docx`。下面导出我们的文档看看。

```bash
$ pandoc \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -M figPrefix="图" \
  -M figureTitle="图" \
  -M tblPrefix="表" \
  -M tableTitle="表" \
  -M eqnPrefix="公式" \
  -N \
  --reference-doc=/home/chunshuyumao/Documents/Pandoc/paper.docx
  --mathjax=/home/chunshuyumao/Documents/Pandoc/MathJax/es5/tex-mml-chtml.js \
  --filter pandoc-crossref \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.docx
```

![按照本业论文格式改的效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120161832.png)

大功告成！现在 Pandoc 已经被驯服了。不过这还不算完善，每次要写这么长的命令，鬼才乐意。接下来就是如何优化的 Pandoc 配置。

## 优化 Pandoc 配置

优化配置还得看官网。官网说我们可以使用一个 YAML 格式配置选项和 `metadata` 然后让 Pandoc 读取。YAML 是什么不用担心，就是一个标记语言，是 "YAML Ain't a Markup Language" 的递归缩写。这是开源世界的一种玩笑，很明显这就是一个标记语言，所以实际上它叫做 "Yet Another Markup Language"。YAML 可以清晰明了地展示配置。闲话少叙，直接看这是如何使用.

![配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120162426.png)

![Pandoc 配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120163224.png)

官网上有如何配置和默认的配置信息，我们只需要修改一些即可。首先祭出我们又臭又长的命令行参数：

```bash
$ pandoc \
  -M title="Convert from Markdown to HTML" \
  -M link-citations=true \
  -M figPrefix="图" \
  -M figureTitle="图" \
  -M tblPrefix="表" \
  -M tableTitle="表" \
  -M eqnPrefix="公式" \
  -N \
  --reference-doc=/home/chunshuyumao/Documents/Pandoc/paper.docx
  --mathjax=/home/chunshuyumao/Documents/Pandoc/MathJax/es5/tex-mml-chtml.js \
  --filter pandoc-crossref \
  --citeproc \
  --bibliography /path/to/lib.json \
  --csl /path/to/style.csl \
  test.md \
  -o test.docx
```

### MetaData YAML 文件

`-M` 开头的都是 `metadata` ，所有创建一个 `meta.yaml` 文件，把它们写进去，`#` 开头的是注释，方便注释参数的作用。YAML 使用冒号而不是等号，而且冒号之后还得有一个空格。

```yaml
# -------------------------------------------------------------
# Pandoc-crossref
# -------------------------------------------------------------

figPrefix: "图"
figureTitle: "图"

tblPrefix: "表"
tableTitle: "表"

eqnPrefix: "公式"

# 支持交叉引用跳转
linkReference: true

# -------------------------------------------------------------
# Division
# -------------------------------------------------------------
chapters: true


# -------------------------------------------------------------
# Citations
# -------------------------------------------------------------
link-citations: true
# 不需要将参考文献做成连接
link-bibliography: false
# 参考文献
reference-section-title: "参考文献"

# 这是 HTML 字体大小。默认字体太大了，改小一点，15、16 均可
fontsize: 14px
```

这里多了之前没有讲的东西。首先是 `linkReference`，用于支持交叉引用跳转，默认没有开启，需要配置。`link-bibliography` 不让 Pandoc 给参考文献加上链接。这个说起来比较抽象，图片伺候：

![加链接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120164411.png)

![不加链接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120164244.png)

最后一行是参考文献标题大名。有人可能发现了，`title` 没有出现在 meta.yaml 里。确实，因为没有必要，除了将 `metadata` 写到 meta.yaml 外还可在 Markdown 文件中直接写，使用 `---` 隔开即可，这叫做 `front-matter`(标头)，比如我们的原文件：

![标头](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120171326.png)

除了这些，所有的 `-M` 的属性都可以写到 front-matter 中。

### Options YAML 文件

写完 `metadata` 文件来看选项文件。命令行中所有非 `-M` 都是选项。注意，之前转换格式时少了一点，就是显式指定格式。实际上我们是从 Markdown 格式 转成 HTML 或者 DOCX 的。这次为了规范，我们分别为两者指定各自的文件，先写 DOCX 的 YAML 文件：

```yaml
# 指定从 Markdown 格式开始转换
reader: markdown
# 指定转换成的格式是 DOCX
writer: docx

# -------------------------------------------------------------
# Reader Options
# -------------------------------------------------------------
# 设置 Tab 为 4 个空格
tab-stop: 4
preserve-tabs: false

# 使用 pandoc-crossref 作为交叉引用
filters:
  - pandoc-crossref
  - citeproc

# 指定 metadata 从 meta.yaml 中读取
metadata-files:
  - meta.yaml

# 设置标题偏移。例如将一级标题变成三级标题，那就改成 2
shift-heading-level-by: 0
# 设置图片默认扩展名。如果自己太懒连格式都没写，那就让 Pandoc
# 自动帮你添加 —— 那你是真的懒
default-image-extension: .png

# -------------------------------------------------------------
# Writer Options
# -------------------------------------------------------------
# 这是设置默认结束符，Windows 可以设置为 nl
eol: lf
# 是否自动换行。如果一行太长了，Pandoc 可以帮助自动换行
wrap: auto

# 是否生成目录。
toc: false
# 生成目录的话，生成多深的目录，这里只需要生成二级目录就可以了
toc-depth: 2

# 是否是单文件模式，DOCX 可以省略，但是 HTML 最好显式写下来
standalone: true
# 是否去掉注释。这指的是写 Markdown 时使用的 HTML 类型的格式，例
# 如 <!--这是注释，没必要保留到正文-->
strip-comments: true

# Fore reference-doc, metadata-files
# 设置查找 meta.yaml 和 模板文件的路径，设置之后就不用写很长的路
# 径了，不信看看上面的 meta.yaml 压根就不用写路径，这里已经设置了
data-dir: /home/chunshuyumao/Documents/Pandoc

# For bibliography, and file resource
# 查找文献库和图片的路径
resource-path:
  - .
  - /home/chunshuyumao/Documents/Pandoc


# -------------------------------------------------------------
# Sepecific writer
# -------------------------------------------------------------
reference-links: true

# document | block | section
reference-location: document

# default | chapter | section | part
top-level-division: chapter

number-offset: []
# 是否自动分章节
number-sections: true
# 实际上，模板找不到，所以这里还是需要写全路径
reference-doc: /home/chunshuyumao/Documents/Pandoc/paper.docx

# -------------------------------------------------------------
# Citation options
# -------------------------------------------------------------
# 文献库，这里就不用写全路径了
bibliography:
  - ZoteroLibrary.json
csl: csl/chinese-gb7714-2005-numeric.csl

```

同理，HTML 的 YAML 文件

```yaml
reader: markdown
writer: html

# -------------------------------------------------------------
# Reader Options
# -------------------------------------------------------------

tab-stop: 4
preserve-tabs: false

filters:
  - pandoc-crossref
  - citeproc

metadata-files:
  - meta.yml

shift-heading-level-by: 0
default-image-extension: .png

# -------------------------------------------------------------
# Writer Options
# -------------------------------------------------------------

eol: lf
dpi: 72
wrap: auto

toc: true
toc-depth: 2

strip-comments: true

standalone: true
# 这里要注意，如果为 true， 每次转换成 HTML 格式，Pandoc 都会尝试
# 下载 Markdown 中的图片。这种如果使用图床比较多，最好直接关了这东
# 东，不然转换很慢的。
embed-resources: false

# Fore reference-doc
data-dir: /home/chunshuyumao/Documents/Pandoc

# For bibliography, images
resource-path:
  - .
  - /home/chunshuyumao/Documents/Pandoc


# -------------------------------------------------------------
# Sepecific writer
# -------------------------------------------------------------

reference-links: true

# document | block | section
reference-location: document

# default | chapter | section | part
top-level-division: chapter

number-offset: []
number-sections: true

# 代码高亮的主题
highlight-style: pygments
# 数学公式引擎
html-math-method:
    method: mathjax
    url: /home/chunshuyumao/Documents/Pandoc/MathJax/es5/tex-mml-chtml.js


# -------------------------------------------------------------
# Citation options
# -------------------------------------------------------------

bibliography:
  - ZoteroLibrary.json
csl: cls/chinese-gb7714-2005-numeric.csl
```

把这三个文件放到 `/home/chunshuyumao/Documents/Pandoc/` 下，形成这样的结构：

![文件结构](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120170716.png)

划红线的表示用不到的，有些只是我写的脚本。`paper.docx` 和 `ref.docx` 是 DOCX 模板。做好后如何用？

```bash
$ pandoc -d ~/Documents/Pandoc/defaults/docx.yaml test.md -o test.docx
```

即可。这便是为啥建议所有东西放一个文件夹，多简单呀。至此，基本配置已经做完。但问题远远没有结束，虽然展示时没有这个毛病，但是写论文一定会碰到，那就是 CSL 文件导出的文献其实是有问题的。举个例子：

![这就是毛病](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120172424.png)

作者多于三个就会使用“等”替代，但是 CSL 文件只能使用一种语言，Pandoc 的分析可能也一样，原本英文文献应该出现“et al”，但中文本地化的前提下它只能使用“等”。这个问题不仅 Pandoc 有，Zotero 也有，主要问题出在使用 Zotero 的 CSL 文件。不用 Zotero 的 CSL 能不能避免这种情况？答案是：不知道。CSL 文件用的是 XML(Extensible Markup Language, 可拓展标记语言)，这种标记语言简直是魔鬼，反正我是没看懂，遑论去改了。

这也不是没救，无法改变 CSL 文件，可以上网找别人改过的。不过我试了很多发现也没用，很可能 Pandoc 也有问题。最后曲线救国，只能写 Pandoc 的 filter 进行修改。这也是为啥我的文件夹有几个脚本。前面大家看到了，Pandoc-crossref 就是一个 Pandoc 的 filter。Pandoc filter 支持 Lua 语言，拓展不难。只是这一篇已经够长了，只能在下一篇讲如何将 Pandoc 渲染出的参考文献改为正常的词。

# 后语

隔了大半年才再写一篇其实也是懒。期间我学五笔输入法，为了适应五笔强迫使用，结果打字速度不行，没法写文章。且最近几个月好像也不得空，期间还换了电脑系统，从 Manjaro 换成了 Arch Linux。中途遇到 Zettlr 出问题，于是从 Zettlr 迁移到 Vim。现在的 Markdown 是用 Vim 写的。当初不知道 Better BibTex for Zotero 可以在 Vim 中直接调用窗口引用文献，我甚至还写了一个 Vim 插件用于文献参考，效果如下

![自己写的 Vim 参考文献补全插件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221120193841.gif)

感觉还行，顺便解决了 Vim 表格问题和图片插入。反正这几个月自己虽然没有写文章，但是确实没有闲着，都折腾需要折腾的东西了。如果最近要写点东西还是有素材的。
