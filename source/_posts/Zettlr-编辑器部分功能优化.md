---
title: Zettlr 编辑器部分功能优化
hide: false
tags:
  - Zettlr
categories:
  - Linux
math: false
mermaid: false
date: 2022-05-10 09:28:40
index_img:
excerpt: 优化 Zettlr 编辑器的部分功能
---

# 前言

因为 Zettlr 编辑器支持在 Markdown 下的文献引用，自己逐步从 Tyopra 转向 Zettlr。和 Tyopra 比，Zettlr 的体验确实不是很好。此外，Zettlr 还存在图片默认左对齐、文章字数统计对中英文支持不好等问题。

![图片左对齐，中英文（右上角）混合统计有问题。按理来说统计有 4 千字，这里的统计接近 8 千字。](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205100936007.png "图片左对齐，中英文（右上角）混合统计有问题。按理来说统计有 4 千字，这里的统计接近 8 千字。")

我希望的是图片可以默认居中对齐，然后中英文混输的字数统计可以准确一点。前者可以通过自定义 <ruby>CSS（层叠样式表）<rt>Cascading Style Sheets</rt></ruby> 进行修改，后者只能修改源码了。

![修改后的效果](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205100942279.png "修改后的效果")

既然要修改源码，索性一并修改 <ruby>CSS（层叠样式表）<rt>Cascading Style Sheets</rt></ruby> 的源码。因此这就是接下来的工作。接下来的修改在 Zettlr 2.2.5 和 2.2.6（最新版） 版本都是通用的。以后的版本可能在代码上有所不同，但本修改应该仍然起作用。由于是自己修改源码，每次发布新版的时候都需要手动修改。一个好的解决办法是给 GitHub 上的维护着提个 issue 或者直接拉取分支和人家一起干（这不是我该干的）。

# 开工

开始之后需要你的电脑安装这几样东西：

1. [Node.js](https://nodejs.cn "Node.js")，让 JavaScript 脱离浏览器。版本什么其实不重要，不要太旧就行。Linux 发行版一般可以通过命令行安装，例如我的 Manjaro ：`sudo pacman -S nodejs --noconfirm`。
2. [Yarn](https://www.yarnpkg.cn "Yarn")，JavaScript 软管理包，用于拉取源码所需的各个包。Linux 发行版一般可以通过命令行安装，例如我的 Manjaro：`sudo pacman -S yarn --noconfirm`。
3. [Git](https://git-scm.com "Git")，分布式版本管理系统，用于从 GitHub 拉取 Zettlr 源码。这个是可选的——因为可以直接从 GitHub 上下载源码，不一定通过 Git 拉取。

Windows 系统需要到官网下载，当然，如果你使用 [Chocolatey](https://chocolatey.org "Chocolatey") 等 Windows 下的包管理器的话也可以选择直接使用这个管理器下载——据我所知，这个包管理器现在已经包含了许多的软件。

## 获取 Zettlr 源码

获取 Zettlr 源码有两种方式，分别是 Git 拉取和 GitHub 直接下载。

1. Git 拉取。

首先创建一个文件夹用于保存 Zettlr 源码，随便取个名字，这里就叫做 Shr2hst。进入文件夹中，并通过<ruby>克隆<rt>clone</rt></ruby>指令拉取源码。

```bash
$ mkdir ~/Shr2hst && cd ~/Shr2hst
$ git clone https://github.com/zettlr/zettlr
Cloning into 'zettlr'...
remote: Enumerating objects: 47557, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 47557 (delta 0), reused 0 (delta 0), pack-reused 47554
Receiving objects: 100% (47557/47557), 105.81 MiB | 1.88 MiB/s, done.
Resolving deltas: 100% (36147/36147), done.
$ ls
zettlr
$ cd zettlr
$ ls
CHANGELOG.md          forge.config.js  scripts      tsconfig.json
CITATION.cff          LICENSE          SECURITY.md  webpack.main.config.js
CODE_OF_CONDUCT.md    package.json     source       webpack.renderer.config.js
CONTRIBUTING.md       README.md        static       webpack.rules.js
electron-builder.yml  resources        test         yarn.lock
```

这个操作一般会比较慢——国内一直比较慢，所以需要耐心。进入 Zettlr 文件夹之后可以看到许多的文件，其中 source 文件夹就是我们的源码所在的位置。

2. GitHub 下载压缩包。

打开浏览器，输入 `https://github.com/zettlr/zettlr` 进入 GitHub 界面，选择 Code 中的 Download ZIP 进行下载。

![](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205101008484.png)

下载完成之后可以进行解压。在 Windows ，随便使用一个压缩软件就可以解压。在 Linux 中可以使用 unzip 命令解压，效果如下。

```bash
$ unzip Zettlr-develop.zip
$ ls
Zettlr-develop  Zettlr-develop.zip
$ cd Zettlr-develop
$ ls
CHANGELOG.md          forge.config.js  scripts      tsconfig.json
CITATION.cff          LICENSE          SECURITY.md  webpack.main.config.js
CODE_OF_CONDUCT.md    package.json     source       webpack.renderer.config.js
CONTRIBUTING.md       README.md        static       webpack.rules.js
electron-builder.yml  resources        test         yarn.lock
```

## 字数统计

首先修改我们的最初目标中英文混合的字数统计。这是一个 TypeScript 文件，位于 source/common/util/count-words.ts。

Windows 用户，建议通过文件管理找到这个文件，然后使用你喜欢的编辑器打开。
Linux 用户可以在命令行上输入 `vim source/common/util/count-words.ts` 直接打开文件进行修改。

转到最后一行，把 44 到49 行的代码删掉或注释掉，在后面添加以下代码。

```bash
  content = content.replace(/(\r\n+|\s+| +)/g, "𰻝");
  content = content.replace(/[\x00-\xff]/g, "w");
  content = content.replace(/w+/g, "*");
  content = content.replace(/𰻝+/g, "");
```

![未更改](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205101026625.png "未更改")

![更改后](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205101030724.png "更改后")

保存退出。这几行代码的统计方式虽然原始，但是十分高效——和 Word、WPS Word 的统计基本是一样的。代码里的“<ruby>𰻝<rt>biáng</rt></ruby>”字如果打不出来可以使用其他生僻字代替，例如“ <ruby>龘<rt>dá</rt></ruby>”“ <ruby>爨<rt>cuàn</rt></ruby>”“<ruby>厶<rt>sī</rt></ruby>”“ <ruby>鬱<rt>yù</rt></ruby>”“<ruby>𣊧<rt>lǎng</rt></ruby>”等等。其实上面的统计很简单，大概意思是：先将空格符、回车符、换行符换成一个生僻字，然后将拉丁字母替换成 w ，在把以生僻字隔离的一个至多个 w 替换成 * ，然后减去生僻字就是所有的字数了。

```bash
# 举个例子，看下面的文字
我 Wǒ 是 shì 爱 aì 南 nán 开 kāi 的 de 。
俺也一样。
# 替换后
我𰻝Wǒ𰻝是𰻝shì𰻝爱𰻝aì𰻝南𰻝nán𰻝开𰻝kāi𰻝的𰻝de𰻝。𰻝俺也一样。𰻝
# 替换拉丁字母
我𰻝ww𰻝是𰻝www𰻝爱𰻝ww𰻝南𰻝www𰻝开𰻝www𰻝的𰻝w𰻝。𰻝俺也一样。𰻝
# 去掉拉丁字母
我𰻝*𰻝是𰻝*𰻝爱𰻝*𰻝南𰻝*𰻝开𰻝*𰻝的𰻝*𰻝。𰻝俺也一样。𰻝
# 去掉生僻字
# 1 2 3  4 5 6 7 8 9 10 11 12 13 14 15 16 17 18
 我 * 是 * 爱* 南* 开*  的 *  。 俺 也 一 样 。 
```

![以防出现方框](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/10/202205101207897.png)

## 图片居中

图片居中的源码位于 source/common/modules/markdown-editor/editor.less 文件中，使用任意文本编辑器打开。转到 217 行——或者慢慢找也行。在 figture 里边的 img 添加 `margin: 0 auto;`，表示居中对齐。保存退出。

```bash
$ vim source/common/modules/markdown-editor/editor.less
```

![未修改](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205101049346.png "未修改")

![修改后](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205101050726.png "修改后")

这一部分是前端知识，知道这么回事就行。

# 编译

编译之前下载依赖包，执行一下命令。

```bash
$ yarn install --frozen-lockfile
```

确认安装完依赖包之后就可以进行编译。

编译选项包括：

* `package:mac-x64` (Intel-based Macs)
* `package:mac-arm` (Apple Silicon-based Macs)
* `package:win-x64` (Intel-based Windows)
* `package:win-arm` (ARM-based Windows)
* `package:linux-x64` (Intel-based Linux)
* `package:linux-arm` (ARM-based Linux)

如果不知道自己的系统是啥构架，直接使用 `yarn package` 就可以了。

```bash
$ yarn package:linux-x64 # 这个是编译 Linux 的包
✔ Checking your system
✔ Preparing native dependencies
✔ Compiling Main Process Code
✔ Compiling Renderer Template
✔ Compiling Renderer Preload: main_window
✔ Compiling Renderer Preload: print
✔ Compiling Renderer Preload: log_viewer
✔ Compiling Renderer Preload: quicklook
✔ Compiling Renderer Preload: preferences
✔ Compiling Renderer Preload: tag_manager
✔ Compiling Renderer Preload: paste_image
✔ Compiling Renderer Preload: error
✔ Compiling Renderer Preload: about
✔ Compiling Renderer Preload: stats
✔ Compiling Renderer Preload: assets
✔ Compiling Renderer Preload: update
✔ Compiling Renderer Preload: project_properties
✔ Preparing to Package Application for arch: x64
✔ Preparing native dependencies
✔ Packaging Application
Done in 236.96s.
```

编译完成，接下来就是打包。

打包选项包括：

* `release:mac-x64` (Intel-based Macs)
* `release:mac-arm` (Apple Silicon-based Macs)
* `release:win-x64` (Intel-based Windows)
* `release:win-arm` (ARM-based Windows)
* `release:linux-x64` (Intel-based Linux)
* `release:linux-arm` (ARM-based Linux)

Linux 系统打包会生成多个安装包，如 deb、rpm、zip 和 Appimage。这个过程可能还会失败。由于 Appimage 包是公用包，几乎所有的 Linux 系统都可以使用，所以我建议按照这个来打包。好处是通用、简单，缺点是有点大。

```bash
$ yarn release:linux-x64 # 生成所有预定义的包
$ yarn electron-builder --linux AppImage --x64 --publish never --prepackaged out/Zettlr-linux-x64 # 只生成 Appimage包
  • electron-builder  version=23.0.3 os=5.15.32-1-MANJARO
  • loaded configuration  file=/root/Shr2hst/zettlr/electron-builder.yml
  • writing effective config  file=release/builder-effective-config.yaml
  • building        target=AppImage arch=x64 file=release/Zettlr-2.2.6-x86_64.AppImage
Done in 8.78s.
```

打包完成后，所有的包都在 Zettlr 根目录下的 release 文件夹中。Windows 系统可以直接运行安装。Linux 系统的 Appimage 包还有点事要做。

# Appimage 包

首先赋予 Appimage 包可执行权限。

```bash
$ chmod u+x Zettlr-2.2.6-x86_64.AppImage
$ mkdir -p ~/.local/bin/
$ cp Zettlr-2.2.6-x86_64.AppImage ~/.local/bin/
$ ln -s ~/.local/bin/Zettlr-2.2.6-x86_64.AppImage  ~/.local/bin/zettlr
$ ./Zettlr-2.2.6-x86_64.AppImage --appimage-extract
$ mkdir -p ~/.local/share/applications/
$ mkdir -p ~/.local/share/icons/
$ cp squashfs-root/zettlr.desktop ~/.local/share/applications/
$ cp squashfs-root/zettlr.png ~/.locals/share/icons/
$ rm -rf squashfs-root
```

输入 `vim ~/.local/share/appications/zettlr.desktop` 修改 

```bash
Exec=AppRun --no-sandbox %U
``` 

为

```bash
Exec=zettlr --no-sandbox %U
```

保存退出。至此，Zettlr Appimage 已经按转完毕，可以直接从桌面启动了。如果是 Windows 就没有这么多的麻烦。

Zettlr 的源码可以删掉也可以保留——爱咋咋地。如果想给 Zettlr 添加功能的话可以自己在源码的基础上修改，我对 Zettlr 还是挺满意的。

# 后语

这篇博客无非就是记录一下自己修改的 Zettlr 的经历（虽然只是改了一丢丢）。这里可以谈谈使用 Manjaro 发现的问题。

记得在写安装常用软件教程的时候我说过自己的 Manjaro 使用不了 Flameshot 截图软件，其实还包括 Shutter、Deepin-screenshot、Wemeet 等等软件。后两者存在的问题是：可以正常使用，但是截图截的是黑屏、共享的也是黑屏。不信邪的我源码编译了 Flameshot，结果发现还是一样无法使用。

最近转念一想：好家伙，原来是 GNOME 4.1 之后的窗口系统后端默认是 Wayland。Wayland 是新生代，有天然的优势，但很多软件都没有支持，所以录屏、截屏甚至共享屏幕都使用的 X11 的接口，这自然就出现黑屏。

GNOME 也考虑到这一点，所以在用户登陆界面的右下角提供了一个后端选择选项，就是一个齿轮模样。其中的选项：

* GNOME，表示默认使用 Wayland 后端。
* GNOME Classic，是经典的 GNOME 后端，不启用各种插件。
* GNOME Xorg，启用 X11 后端。

启用 X11 虽然可以使用我上面提到的软件，但是我发现它不像 Wayland 默认支持<ruby>触摸板手势<rt>Touchpad Gestures</rt></ruby>。这个应该可以通过安装 GNOME Shell <ruby>插件<rt>Extensions</rt></ruby> X11 Gestures 支持操作。反正现在我还是安于 Wayland 后端。

