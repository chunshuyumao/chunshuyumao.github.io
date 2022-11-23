---
title: VSCode 通过 SSH 远程使用服务器
hide: false
tags:
  - SSH
categories:
  - Linux
math: false
mermaid: false
date: 2022-11-22 10:31:21
index_img:
---

# 前言

统计计算和机器学习常常需要借助计算力更强的计算机，也就是服务器。然而服务器一般只能通过 SSH(Secure Shell) 登陆，没有界面。SSH 可以在自己的电脑和服务器之间建立安全的链接，使用自己的服务器。没有界面只有控制台，对于浸淫命令行操作的人来说不是什么大问题；然而很多的科研人员并不是专业开发人员，并不一定对计算机很了解，只有一个终端就成了障碍。

[VS Code(Visual Studio Code)](https://code.visualstudio.com/) 是微软推出的一款跨平台编辑器。因其强大的扩展能力和微软雄厚的资金支持，VS Code 得到快速发展，现在基本成一个可以配置成任何一门语言 IDE(Integrated Development Environment，集成开发环境) 的编辑器。鉴于此，本篇将通过简单的配置，在 VS Code 上使用自己的远程服务器。

# 预备

想使用 SSH 需要自己的电脑安装 SSH。Windows、Linux 和 MacOS 系统一般都自带 SSH。Windows10 以下则需要手动配置。验证自己电脑是否有 SSH，可以打开控制台，输入 `ssh --version`. 如果出现类似的提示则表明本机的 SSH 可以使用。如果不可以用，Windows 系统到 [微软教程](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui) 查看如何安装；MacOS 使用 `brew install openssl` 进行安装，一般预装；Linux 系统没有安装就使用自己的包管理器或到软件应用商店安装。

> Linux 系统的包管理器太复杂了，一般来说
>
> Arch Linux 系列: `sudo pacman -Syu openssl`
>
> Ubuntu/Debian 系列: `sudo apt install openssl`
>
> RHEL 系列: `dnf install openssl` 或者 `yum install openssl`
>
> 这是 Linux 最常见的三个分发版。

![检查 SSH 是否安装成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122110635.png)

SSH 准备好就可以安装 VS Code 了。

# 配置 VS Code

到 [VS Code(Visual Studio Code)](https://code.visualstudio.com/) 官网去下载对应的版本然后安装。安装步骤比较简单，此处不再赘述。安装打开 VS Code 并按照提示安装中文插件，不愿使用中文本地化插件可以不安装。安装完毕之后选择左上角的四个方框的图标，那是插件商店。搜索 SSH，安装 Remote SSH 和 Remote X11，前者用于远程登录，后者用于端口转发。当然，也可以不安装后者。

![安装 SSH 插件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091008.png)

插件安装完毕按 `CTRL+SHIFT+P` 快捷键调出命令面板，输入 ssh 添加主机——也就是服务器。输入 `ssh username@hostip` 添加主机，然后保存。

![调出命令面板](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091126.png)

![添加主机](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091302.png)

![选择保存的文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091340.png)

添加完毕，VS Code 会提示是否连接，这时候可以选择连接。下次想要再使用不需要添加主机，而是直接呼出命令面板，然后输入 ssh connect 即可：

![是否连接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091358.png)

![再次连接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091506.png)

第一次连接，VS Code 会询问主机是什么系统，一般服务器都是 Linux，这里演示的主机服务器是 Linux，所以选择 Linux 系统。连接速度因人而异，期间可能需要输入密码。这里的演示不需要，因为后面可以设置不需要密码登录。

![Linux 主机](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091532.png)

![正在连接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091546.png)

连接成功后会弹出一个提示，告诉你对主机的配置放在何处，不用担心。

![弹出的提示](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122091629.png)

按照 `终端->新建终端` 或者快捷键打开终端。这时可以看到服务器的终端。

# 使用 VS Code

在连接 SSH 的前提下看到左边启动界面有一个打开文件夹的操作，可以选择打开服务器的目录。打开目录时，VS Code 会询问是否信任该目录。勾选信任，不然无法操作。

![打开目录](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092128.png)

![信任](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092204.png)

信任之后可以打开目录下的文件。打开文件时 VS Code 会提示是否安装某些插件，建议直接选择安装这类插件。

![打开文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092305.png)

![安装插件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092330.png)

VS Code 功能非常强大，可以在左边 Git 侧边栏查看修改的文件有哪些。

![Git 侧边栏](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092420.png)

插件安装完毕可以进行其他操作，比如看到 VS Code 提示找不到这些 Python 包。这是因为我们没有激活合适的服务器环境。这里的服务器使用 conda/mamba 进行环境管理，可以手动选择合适的环境。呼出命令面板，输入 `python interpreter` 寻找合适的 Python 环境。我的服务器工作环境是 Pytoch2，因此选择该环境。

![没有找到合适的包](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092553.png)

![呼出命令面板](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092814.png)

![选择环境](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092835.png)

选择合适的环境后，VS Code 可以找到 Python 包了。

![找到软件包](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122092915.png)

# 运行

打开好环境，我们可以尝试运行。选择 VS Code 右上角的运行，点击即可。运行的输出会出现在终端。

![运行](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122093014.png)

![终端输入](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122093042.png)

运行结果如何先不管。但是如果输入有图片，VS Code 也可以查看。

![查看图片](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122093554.png)

# 关闭

当运行完毕，没有其他工作了，可以选择关闭 SSH 连接。呼出命令面板，输入 `close remote connection` ，确认即可。

# 后语

简单记录一下罢了。除了 VS Code，其实还可以使用 Jupyter。不过自己的工作环境安装 Jupyter 有问题，除非新建环境。既然如此，那又何必折腾？虽然 Jupyter 看起来确实漂亮。

# 补充

如何使用 SSH 密钥直接登陆服务器，不用每次都输入密码？

在自己的终端输入

```bash
$ ssh-keygen -r rsa
```

回车三次。不需要输入啥。这一步会在自己家目录 `.ssh` 下生成两个密钥文件: id_rsa 和 id_rsa.pub，前者是私钥后者是公钥。公钥可以部署到服务器用于配对，私钥用于确认身份。私钥不可公开，否则就没有意义了。

![生成密钥对](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221122124637.gif)

生成密钥对之后在命令行输入

```bash
$ ssh-copy-id user@host
```

`user` 是用户名，`host` 是服务器 IP。这会默认复制你的公钥到服务器 `~/.ssh/authorized_keys` 文件中。做完这些，以后登录服务器就不必输入密码了。
