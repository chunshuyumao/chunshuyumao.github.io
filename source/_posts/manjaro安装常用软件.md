---
title:  Manjaro 安装常用软件
tags:
  - 应用
categories:
  - Linux
math: false
mermaid: false
date: 2022-03-05 11:05:59
index_img:
excerpt: 安装完了 Manjaro，接下来就是安装常用的应用了。
---

# 前言

前面我们已经安装好  Manjaro 系统了，现在就是安装一些常用的软件。此外，自己并非工科生，使用的 Windows 专有软件不多。

---

# 配置国内源以及更新系统

  Manjaro 是国外的系统，服务器往往在国外。想要使用  Manjaro 官方软件包需要配置。以前使用 Arch 需要手动配置国内源，然而  Manjaro 更加简单：它把世界各地的软件源镜像组合到官方的文件源系统，可以使用简单的工具自动为你筛选速度最快的软件源。

首先打开你的终端，输入

```shell
sudo pacman-mirrors -c China -m rank
```

其中，`sudo`允许你行使管理员的命令，`pacman-mirrors`是选择最快源的命令；`-c`是指定地区，这是使用中国；`-m`是方法，我们选择`rank`，就是把速度最快的挪到前面。它会要求你输入你的密码，注意，输入密码的时候你是看不到反馈的——也就是说，你看不到自己输入的密码，那里黑漆漆的一片。

![配置国内源](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051124391.png)

![配置完成](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051126408.png)

好的，现在你已经有了对你来说最快的软件源。接下来我们先更新系统：

```shell
sudo pacman -Syyu
```

当它询问是否安装的时候，我们选择 是 。等它继续更新系统，再次选择 是 。

![更新系统，选择y](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051127397.png)

在它安装这个阶段，我先介绍一下我们的指令是什么意思。

  Manjaro 使用的是 pacman 包管理器，记住 pacman 的一些指令即可：

```shell
# 这是安装一个包
sudo pacman -S 包名

# 这是查询一个包
pacman -Ss 包名

# 这是删除一个包
sudo pacman -R 包名

# 这是删除一个包和它的依赖包
sudo pacman -Rs 包名

# 这是删除系统不使用的、多余的包
sudo pacman -Rs $(pacman -Qtdq)
```

如果你还想知道`pacman`的更过用法，你可以使用：

```shell
man pacman
# 或者
pacman --help
# 或者
pacman -h
```

好的，我这边的安装已经完成了。

---

# 安装软件

## 安装官方应用

### 安装截图软件 Flameshot

```shell
sudo pacman -S  Flameshot
```

![ Flameshot效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051234925.png)

  Flameshot 功能非常强大，可惜我的电脑一直运行不了。

  Flameshot 没有全局快捷键，可以去 设置 里设置快捷键。这些教程可以到网上查。

### 安装多线程下载器

 axel 是优秀的多线程下载工具——命令行。如果你习惯使用命令行下载， axel 绝不能被错过；如果你不想使用命令行，那就下载 XDM ：

```shell
sudo pacman -S axel
# 或者
sudo pacman -S xdm
```

 axel 的使用：

```shell
axel -n 15 address
```

其中`-n`后面紧跟的是连接数，理论上连接数越多越好，但是不建议太高。`addrss`是下载地址。

 XDM 的使用：

 XDM 和 NDM 、 IDM 一样，都可以嵌入浏览器，然后自己嗅探下载的东西。

### 安装邮件客户端雷鸟(Thunderbird)

如果使用邮件客户端，建议使用雷鸟，毕竟它还是很优秀的。

```shell
sudo pacman -S thunderbird
```

安装完雷鸟之后，还可以在它的插件中安装 Markdown Here 插件，实现 Markdown 写邮件。

### 安装virtualBox虚拟机

首先查看你的内核版本：

```shell
uname -r
```

![内核版本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051257678.png)

然后选取和自己内核版本一样的虚拟机，我这里的内核是 15 ，所以选择`6`。

![安装指定版本虚拟机](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051258261.png)

### 安装视频播放器VLC

 VLC 就像 Windows 下的 PotPlayer ，所以建议安装。

```shell
sudo pacman -S vlc
```

### 安装 Markdown 编辑器(选择性) 

```shell
sudo pacman -S zettlr
```

### 安装vim(选择性)

 Linux 下，谁人不知 VIM 编辑器？

```shell
sudo pacman -S vim
```

### 安装中文输入法

输入以下命令后，直接回车，默认选中所有的输入设置。

```shell
sudo pacman -S fcitx5-im fcitx5-chinese-addons
```

![安装中文输入法](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051336705.png)

安装完毕，开始配置中文输入法。输入

```shell
sudo vim /etc/environment
```

回车，按住<kbd>Shift</kbd>+<kbd>g</kbd>跳到文件末尾。按下<kbd>o</kbd>在末尾添加以下配置

```shell
INPUT_METHOD="fcitx"
GTK_IM_MODULE="fcitx"
QT_IM_MODULE="fcitx"
XMODIFIERS="@im=fcitx"
```

然后按<kbd>Esc</kbd>键，输入

```shell
:wq
```

回车，保存退出VIM。

![保存退出VIM](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203162212673.png)

输入`reboot`重启系统。

![重启](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051318337.png)

重启之后，我们可以看到右上角多出一个输入法配置。点击它进行配置。

![输入法](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051339358.png)

切换中文和英文输入法的快捷键是<kbd>Shift</kbd>或者<kbd>ctrl</kbd>+<kbd>space</kbd>。当然也可以改。

![尝试使用中文输入法](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051340640.png)



### 安装其他应用

```shell
# Steam
sudo pacman -S steam
# peek GIF制作
sudo pacman -S peek
```

## 安装AUR应用

### 安装yay

 yay 是  Manjaro 一个非官方的包管理器，使用方法和 pacman 一样，它们的区别是 pacman 是官方的产品。前面提到了，  Manjaro 拥有强大的社区支持。官方的软件虽然安全，但是毕竟能力有限。社区就发展出一个自己的软件源 AUR(Arch User Repository，Arch用户仓库) [^1]。 yay 就是从 AUR 安装软件的工具。

```shell
sudo pacman -S yay base-devel
```

安装完 yay 之后，由于我们下载的软件可能会在 GitGub 上(例如后面的 WPS 的配件)，所以配置一下 GitGub 的 DNS ，毕竟国内上网不容易。

输入

```shell
sudo vim /etc/hosts
```

回车，按住<kbd>Shift</kbd>+<kbd>g</kbd>跳到文件末尾。按下<kbd>o</kbd>在末尾添加以下配置

```shell
140.82.114.4 github.com
199.232.69.194 github.global.ssl.fastly.net
```

然后按<kbd>Esc</kbd>键，输入

```shell
:wq
```

回车，保存退出VIM。

![设置DNS](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051354253.png)

### 安装QQ

 QQ 原生的 Linux 版太古老，看起来像是上古时期的东西，所以我们安装国产系统 Deepin 基于 WINE 的 TIM ，安装的时候可以选择仓库或者直接回车：

```shell
yay -S deepin-wine-qq
# 或者
yay -S deepin-wine-tim
```

遇到对比，可以直接选择不显示( N ):

![选择N](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051230824.png)

下载完成之后启动并配置：

![TIM安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051240756.png)

![TIM登陆](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051241358.png)

### 安装微信

```shell
yay -S deepin-wine-wechat
```

安装微信和 TIM 差不多。由于基于 WINE ，这些应用的体验和在 Windows 上没多大区别。

### 安装WPS

办公软件我还是习惯 WPS 。 WPS for Linux 还没有广告。

```shell
yay -S wps-office wps-office-mui-zh-cn ttf-wps-fonts
```

### 安装typora

正常情况下，使用 yay 是没有办法安装 Typora 的。如果你安装成功了，那就跳过这步；否则，请继续往下阅读。

请确保你已经安装了 debtap 。

```shell
# debtap格式软件转换器
yay -S debtap
```

下载 Typora [^5]。

下载完成之后，我们使用`ls`查看当下目录的软件：

![查看typora](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051422846.png)

输入

```shell
sudo debtap -u
sudo debtap -q *.deb
```

这里我们使用匹配模式，使用`*`代替了`typora`。`-u`是更新 debtap ，`-q`是静默模式。如果不出意外，你的目录下会出现一个压缩包：

![重建之后的压缩包](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051432479.png)

输入

```shell
sudo pacman -U *zst
```

安装 typora 。

![安装完成](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051434525.png)

### 安装坚果云

```shell
yay -S nautilus-nutstore
```

其中选择`nutstore-experimental`

![坚果云](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051445504.png)

### 安装其他软件

```shell
# 网易云音乐
yay -S netease-cloud-music
# vscode 文本编辑器，编程使用，选择性
yay -S visual-studio-code-bin
# 谷歌浏览器，随意
yay -S google-chrome
# zotero文献管理器
yay -S zotero
# 百度网盘 安装百度网盘之前先使用 sudo pacman -S patch
yay -S baidunetdisk-bin
# 安装picgo
yay -S picgo-appimage
# 腾讯会议，如果下载不了，建议按照Typora的方式，到官网下载deb格式之后转化安装
yay -S wemeet
```

至此，我们所有的需要的软件都安装完了。

---

# 美化桌面

先看一下我的主题

![亮色](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051526958.png)

![暗色](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051525247.png)

![图标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051526518.png)

首先，在自己家目录下创建文件夹`.themes`和`.icons`

```shell
mkdir .themes
mkdir .icons
```

安装主题有两种方法

方法一、自己下载

1. 首先下载 WhiteSur Gtk Theme [^2]主题， McMojave-circle [^3]图标， macOS Big Sur [^4]光标
2. 把主题挪到`.themes`，图标和光标挪到`.icons`

方法二、使用我提供的压缩文件

1. 登陆 ftp 

   ```shell
   ftp 101.200.84.36
   ```

2. 用户名是`ftp`，没有密码直接回车。然后下载我提供的压缩包：

   ```shell
   get themes.tar.gz
   get icons.tar.gz
   ```

3. 解压缩

   ```shell
   tar zxf themes.tar.gz -C ~/.themes
   tar zxf icons.tar.gz -C ~/.icons
   ```

然后结束准备，开始安装。

查看自己的  Manjaro 是否有 gome-tweaks (图标显示为 tweaks 或者 优化 )，如果没有就下载：

```shell
sudo pacman -S gnome-tweaks
```

打开软件，按下面配置：

![tweaks配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051614226.png)

![tweaks配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203051615629.png)

如果使用 firefox 浏览器，安装 firefox 主题

```shell
cd ~/.themes/WhiteSur-gtk-theme/
./tweaks.sh -f
```

完结！！

---

## 后语

  Manjaro 作为我的主力系统已经有一段时间了，感觉并非得使用 Windows 。通过一些烈的美化之后，其实  Manjaro Gnome 还是很好看的。虽然与  Manjaro KDE 的高度定制性相比有点小巫见大巫，但是我喜欢的本来就是它的简洁。

使用这些软件也并非一帆风顺，例如我的电脑及使用不了  Flameshot ，可能因为分辨率太高导致字体怎么调都怪怪的等等，但是最终都克服了这些问题。我的电脑没有安装QQ或者微信，因为我感觉自己用不上它们。

腾讯会议不能使用 聊天 ，也看不到 弹幕 。

---

# 相关网站

[^1]: https://aur.archlinux.org/
[^2]: https://www.gnome-look.org/p/1403328
[^3]: https://www.pling.com/s/Gnome/p/1305429
[^4]: https://github.com/ful1e5/apple_cursor
[^5]: https://www.123pan.com/s/gqA9-X7YB
