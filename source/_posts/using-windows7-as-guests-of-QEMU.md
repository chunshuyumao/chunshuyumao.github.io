---
title: 在 QEMU 中安装 Windows7 虚拟机
hide: false
tags:
  - QEMU
categories:
  - Linux
math: false
mermaid: false
date: 2023-04-14 14:25:13
---

# 前言

{% post_link after-a-year-of-using-linux-as-the-main-computer '使用 Linux 作为主力机一年后' %} 提到可以使用 [QEMU](https://www.qemu.org "QEMU") 虚拟一个 Windows 系统，在里边使用国产毒瘤，干净又卫生，今天它来了。

本次教程主要虚拟的是 Windows7，因为 WindowsXP 太老了，自己不知道怎么安装；Windows10 太高级了，装了卡得不要不要的；至于 Windows8 这种系统单纯就是觉得太垃圾了，没必要安装。

开始之前，我们需要一些东东：

1. QEMU, Quick EMUlator. 按照组词习惯，我一般读作 /kju: 'emju/. QEMU 是一个开源的虚拟机和模拟器。据[官网](https://wiki.qemu.org/Main_Page "QEMU Wiki")所说，它可以模拟不同架构的机器和拥有可以媲美原生系统性能的虚拟能力。
2. [KVM](https://www.linux-kvm.page/Main_Page "KVM"), Kernel-based Virtual Machine. 基于 Linux 内核的虚拟机。KVM 内建于 Linux, 一般作为模块（Modules）编译，是高效利用宿主机资源的开源虚拟化技术，使用 QEMU 一般会开启 KVM 提供更好的性能。
3. Windows7 系统镜像。虚拟一个 Windows 系统自然需要一个镜像。现在的 Windows7 似乎已经无法在微软官网上找到了，所以推荐到 [MSDN I tell you](https://msdn.itelyou.cn "MSDN I tell you") 下载原版镜像。这个 MSDN 与微软的 MSDN 不同，它只是一个提供了微软系统原版镜像和各种工具下载的网站。
4. [VirtIO](https://wiki.libvirt.org/Virtio.html "VirtIO"), 一个提供虚拟机优秀读写性能的虚拟化标准，包括磁盘读写和网络传输等。要是没记错应该是红帽公司（RedHat）设计以提高虚拟机性能的，但是忘了出处。
4. 一台 Linux 电脑。我使用的是 Arch Linux, 本身已经将 KVM 编好成模块。

以上便是需要准备的东西了。

# 安装 Windows7 虚拟机

## 下载 Windows7 镜像

到 MSDN 选择系统镜像，找到 Windows7 镜像，选择合适的版本和语言，点开详细信息复制下载链接，用迅雷下载，迅雷刚开始可能没有速度，但是可能有些时候会碰上狗屎运，变得很快。请不要下载百毒上的 Windows7 镜像，因为那些一般都是 Ghost 镜像，需要使用 WinPE 安装。Windows7 不需要专业版或者旗舰版，毕竟只是为了安装一些毒瘤而已，能用就行，不必挑剔。

![MSDN 上的 Windows7 镜像。这些都是原版镜像，与官网一样，甚至安装完你还需要激活。可以选择中文版本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414150441.png)

## 下载 QEMU 和 VirtIO

### QEMU

使用自己的 Linux 系统的安装 QEMU. Arch Linux 上的 QEMU 在 7.0 后对包进行了[分割](https://archlinux.org/news/qemu-700-changes-split-package-setup/ "QEMU change")，分为 qemu-base、qemu-desktop 和 qemu-full. 为了尽量少的安装软件，我选择了安装 qemu-base, 基本满足使用即可。此外 qemu-base 只有一个 qemu-ui-spice 对系统进行图形操作，所以我还安装了 qemu-ui-gtk 包使用图形界面。这些信息可以通过查找自己系统的软件包查看信息。

![Arch Linux 上对 QEMU 分包的消息](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414152322.png)

![使用 pacman -Si 查看 qemu-base 包含的包，只有一个支持 spice 协议的图形界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414152045.png)

![qemu-ui-gtk 的详细信息](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414152224.png)

Arch Linux 的安装命令如下：

```bash
$ sudo pacman -Syu --noconfirm qemu-base qemu-ui-gtk
```

如果不想使用系统仓库编译的 QEMU 也可以去 QEMU 官网直接下载源码编译。QEMU 很小的，编译也很快，可以根据官网的教程进行编译。

![QEMU 官网介绍](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414152738.png)

待下载完成再下载 VirtIO. 

### VirtIO

VirtIO 一般 Linux 系统会编译成模块启用即可，但是 Windows 没有将 VirtIO 编入内核，需要下载额外的镜像安装驱动才行。这里说的下载 VirtIO 其实是下载 Windows 系统需要的 VirtIO 驱动。直接到 [GitHub 仓库](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md "VirtIO 驱动") 下载驱动即可。

## 安装虚拟机

### 准备工作目录

下载完所有东西后打开 Linux 命令行，新建一个工作目录，把需要的东西放好：

```bash
$ mkdir ~/Downloads/QEMU && cd ~/Downloads/QEMU
$ mkdir -v ISOS VirtIO Win7
mkdir: created directory 'ISOS'
mkdir: created directory 'VirtIO'
mkdir: created directory 'Win7'
$ tree
.
├── ISOS
├── VirtIO
└── Win7

4 directories, 0 files
❯ tree
.
├── ISOS
│   └── w7-64bit.iso
├── VirtIO
│   └── virtio-win-0.1.225.iso
└── Win7

4 directories, 2 files
```

![查看工作目录，QEMU 目录下创建 ISOS、VirtIO和 Win7 目录](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414154956.png)

上面是我已经建好的目录，接下来的启动文件主要放到 Win7 文件夹中。

### 创建 Windows 磁盘

开始之前我们需要准备一个 Windows 磁盘用于安装系统。在命令行上输入

```bash
$ cd Win7
$ pwd
/home/chunshuyumao/Downloads/QEMU/Win7
$ qemu-img create -f qcow2 windows7.qcow2 40G
Formatting 'windows7.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=42949672960 lazy_refcounts=off refcount_bits=16
$ ls
windows7.qcow2
```

![进入 Win7 目录并创建一个 40G 大小的磁盘，命名为 windows7.qcow2](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414155509.png)

后面的工作目录都在 Win7 中。使用 `qemu-img` 创建一个磁盘，大小是 40G. 这个大小随意，30G 也是可以的，使用 qcow2 格式是可以调整大小，也就是后边如果觉得 40G 太大了太小了都可以调整。这里给予 40G 是为了后边分出一个数据盘，因为有些操作不允许在 C 盘（系统盘）上进行，所以开个新的盘作为系统盘即可。

>
> qcow2 是 QEMU 上常用的磁盘格式，支持同时读写（copy-on-write）。不用担心给的空间太大，qcow2 是根据你的使用量使用空间的。举个例子，给了 40G qcow2 格式的空间，其实这个磁盘没有占用那么多空间，如果你只用到 1G，那这个磁盘就只有 1G 大小，给的 40G 表示的是虚拟机能用的最大空间，不是你直接划分这么多给虚拟机。
> 

### 编写启动脚本

在 Wind7 目录下编写一个新的启动脚本：

```bash
$ pwd
/home/chunshuyumao/Downloads/QEMU/Win7
$ touch setup.sh
$ ls
setup.sh  windows7.qcow2
$ vim setup.sh
#!/bin/env bash
# encoding: utf-8
# ========================================================
#
# File: setup.sh
#
# Author: chunshuyumao
#
# Description: 
#
# ========================================================

declare options=''

options="${options} -name 'windows7'"
options="${options} -m 1G"
options="${options} -enable-kvm"
options="${options} -cpu host,hv_relaxed,hv_spinlocks=0x1fff,hv_vapic,hv_time"

options="${options} -boot order=dc"
options="${options} -vga virtio"
options="${options} -display gtk,grab-on-hover=on,show-menubar=off,zoom-to-fit=off"
options="${options} -rtc clock=host,base=localtime"
options="${options} -smp 1,sockets=1,cores=1,threads=1,maxcpus=1"
options="${options} -drive file=/home/chunshuyumao/Downloads/QEMU/ISOS/w7-64bit.iso,index=1,media=cdrom"
options="${options} -drive file=/home/chunshuyumao/Downloads/QEMU/VirtIO/virtio-win-0.1.225.iso,index=2,media=cdrom"
options="${options} -drive file=/home/chunshuyumao/Downloads/QEMU/Win7/windowns.qcow2,index=3,media=disk,if=virtio,aio=native,cache.direct=on"
options="${options} -machine usb=on"
options="${options} -device usb-tablet"
options="${options} -device virtio-net,netdev=vmnic"
options="${options} -netdev user,id=vmnic,smb=/home/chunshuyumao/Downloads"

/bin/taskset -c 9-11 /bin/qemu-system-x86_64 ${options}
unset options

```

![编写脚本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414163402.png)

setup.sh 脚本是后面编写的主要部分。下面一行一行解析：

1. 首先，声明了一个变量叫做 options, 后面主要用于编写 QEMU 的选项。
1. 给系统取名字叫做 windows7. 这里取名最好不要有空格。
1. `-m 1G` 是给予电脑的内存大小，1G 足矣。
1. `-enable-kvm` 用于启用 KVM.
1. `-cpu` 那一行，`host` 表示虚拟机使用的 CPU 和主机一样，后面一连串 `hv` 开头的选项是参考 [Arch Wiki](https://wiki.archlinux.org/title/QEMU#Improve_virtual_machine_performance "hv_ options")，据说可以提高 Windows 系统的性能。
1. `-boot` 一行选择了启动顺序，即先从 CD 启动再试试硬盘。
1. `-vga` 选择的是 virtio，这是显卡，因为使用 VirtIO, 我们就选它。
1. `-display` 那一行选择我们安装的 UI，也就是 GTK. 后面的选项是这个 UI 提供的选项。第一个表示启动后之后直接让虚拟机锁定鼠标，这样鼠标就被限制在虚拟机内部了。第二个选项是不显示窗口的菜单栏，因为菜单栏占用的空间太多了。第三个选项则是关闭界面自适应，不然窗口会自动伸缩。我们可以在虚拟机里调整虚拟机的分辨率，不需要它自己改变。
1. `-rtc` 这是 Real-Time Clock，实时时钟的缩写，用来修正虚拟机的时间。选择和宿主一样即可。
1. `-smp` 设置虚拟机的 CPU 个数。选择一个 CPU 即可。后面的参数刻有可无。
1. `-drive` 用于配置镜像，一个是 Windows7 镜像，一个是 VirtIO 驱动，一个是 Windows7 的硬盘/磁盘。前两个后面可以注释掉。
1. `-machine` 用于开启 USB, 也就是可以直接使用宿主鼠标。
1. `-device usb-tablet` 用于确定鼠标位置。
1. `-device virtio-net,netdev=vmnic` 给电脑配置一个 VirtIO 网卡。
1. `-netdev` 用于配置 [Samba](https://www.samba.org "Samba") 位置。也就是虚拟机可以通过 Samba 访问主机的内容，`smb` 后的地址可以改成自己的地址，例如让虚拟机访问用户主目录 `smb=/home/chunshuyumao`. 相当于共享目录。记住，宿主机要安装 Samba，例如 Arch Linux 使用 `sudo pacman -Syu samba` 进行安装。
1. `/bin/qemu-system-x86_64` 用于启动虚拟机。前面的 `/bin/taskset` 是限制虚拟机使用的 CPU 是哪几个。如果 Linux 系统很繁忙，负载过高会随即杀掉几个程序，而且 Linux 系统的任务一般都是随机跳跃的，很可能会导致 12 个 CPU 里只有一个是繁忙的其余都是空闲的（夸张了）。所以我们把自己的虚拟机限定在这些 CPU 上工作，避免意外被系统杀掉。使用绝对路径是为了避免一些不必要的麻烦，其实也可以直接使用 `qemu-system-x86_64` 和 `taskset`。
1. 最后用 `unset` 删掉我们创建的变量。

上面是我虚拟机使用的命令，比较多。QEMU 支持的命令很多，可以通过 `man qemu` 查看相关的命令和说明，或者直接到官网上查看说明书也行。

### 启动安装 Windows 系统

首先赋予脚本可执行权限，然后启动脚本，按照提示进行安装。

```bash
$ chmod +x setup.sh
$ ls
setup.sh  windows7.qcow2
$./setup.sh
```

![赋予脚本可执行权限，并启动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414165158.gif)

![选择键盘和语言安装 Windows7](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414165652.png)

![选择立即安装，此时是载入界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414165751.png)

![同意用户协议什么的](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414165851.png)

![选择自定义安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414165939.png)

选择自定义安装后进入安装界面，但是记住，我们说过 Windows 没有 VirtIO 的驱动，所以此时它并没有正确识别我们的硬盘，我们需要安装驱动。此时它的界面如下：

![Windows 系统无法正确识别硬盘](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414170112.png)

选择加载驱动器，浏览 virio-win 镜像，然后在 amd64 里找到 w7 选择即可。添加之后点击下一步进行安装。

![给 Windows7 安装 VirtIO 驱动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414170254.gif)

等驱动安装完成，VirtIO 硬盘就会被 Windows 正确识别。直接进行下一步安装:

![VirtIO 硬盘被成功识别](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414170817.png)

![安装系统，安装过程需要点时间](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414170911.png)

安装过程比较久，可以先到浏览器下载需要在 Windows7 里安装的软件，例如 QQ 和微信的安装包。下载的时候要选择 Windows 安装包，因为待会要安装到 Windows 虚拟机里。下载好之后可以把安装包移动到 Samba 共享目录下。我的 Samba 目录是 `/home/chunshuyumao/Downloads`, 这样浏览器下载的东西可以直接从虚拟机中看到，虚拟机也可以直接把需要交流的文件放到该目录下分享给宿主机。

![重新启动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414173220.png)

安装完成后的重新启动，请不要再在启动界面按任何按键，否则会进行重新安装！如果不知道这是什么意思，那就在出现上面重新启动界面后什么也不要动，直到出现下面的界面：

![自动重启后的界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414173349.png)

重启后仍会进行安装，因为安装没有彻底完成，还需要一些步骤。不过也不需要碰什么了，只需要看就行了，喝杯茶什么的。

![完成安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414173546.png)

安装完成后设置用户名和密码，密码可以不用设置直接下一步，也可以设置密码。我觉得不设密码没啥问题。密钥直接跳过。对于更新设置，随便，我觉得选以后再询问也可。后面可以按照提示一步一步完成设置。

![设置用户名](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414191438.png)

![设置虚拟机密码，无所谓，可以直接下一步](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414191537.png)

![添加密钥，直接选择下一步，后面再激活](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414191617.png)

![更新设置，选择以后再询问](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414191701.png)

![按提示继续选择即可](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414191724.png)

![完成安装并成功启动！](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414192156.png)

### 安装 VirtIO 网络驱动

因为网络设备我们选择了 VirtIO, Windows 系统一样无法正确识别，所以我们需要安装 VirtIO 的网络驱动。

![Windows 系统无法识别 VirtIO 网络设备，需要安装驱动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414192439.png)

按 Windows 键搜索 设备管理器 ，选择 其他设备 -> 以太网控制器 进行更新。更新程序时选择挂载的 VirtIO CD 作为查找目录，查找之后会提示是否安装，选择安装即可。

![打开设备管理器，选择以太网设备准备安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414192705.png)

![右键选择更新驱动程序](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414192742.png)

![选择浏览计算机以查找驱动程序软件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414192827.png)

![浏览挂载的 VirtIO CD, 选择它进行驱动查找](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414192933.png)

![正在查找驱动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193028.png)

![选择安装驱动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193052.png)

![安装驱动完成，可以上网了](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193331.png)

![选择网络模式，就按自己的选即可，我选择家庭模式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193401.png)

安装完之后就自动连上了网络，可以上网了。不过 Windows7 的 IE 浏览器很古老，没必要使用，可以通过宿主机下载完东西后直接在虚拟机使用。

### 挂载 Samba 目录

记得我们启动虚拟机的时候给它分享了一个目录，我们需要挂载起来。打开文件管理器，在 计算机 右键选择 添加一个网络位置 ，选在自定义位置后在地址栏输入 `\\10.0.2.4\qemu`，并给这个位置取个名字。这个地址是添加 Samba 服务的默认地址，具体可以查看 QEMU 的帮助。

![在“计算机”上右键添加网络位置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193825.png)

![选自自定义位置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193916.png)

![在地址栏输入 `\\10.0.2.4\qemu`, 这是 Samba 的默认地址](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414193950.png)

![随意给分享目录一个名字，这里我不修改，直接保留默认名字](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414194028.png)

![挂载 Samba 目录完毕](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414194439.png)

挂载完毕，可以直接看到共享目录的文件。

### 激活 Windows7 系统

准备完毕后可以选择激活系统。我的分享目录下有一个 `win7active.zip` 文件，里边有激活环境需要的东西。可以直接在共享目录下双击打开，但是有些文件不行，可以通过复制到虚拟机的其他位置再解压或者安装。

![直接在共享目录打开压缩包](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414195121.png)

解压后随便运行其中的 licence1 或者 licence2, 这里我选择了 1, 激活后会直接重启。

![选择确定激活](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414195226.png)

重启后打开控制面板可以看到系统已经激活了：

![控制面板查看系统是否成功激活](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414195606.png)

完成激活后可以进行个性化设置，比如觉得默认分辨率太小可以选择更高的分辨率。

### 分区

前面我们说了，有些操作 Windows 是不让在系统盘进行的，所以我们可以给电脑来个分区，分出一个盘主要用于数据的保存。

首先打开搜索，查找 分区，打开 磁盘管理。可以看到系统盘 C 盘有 40G。我们可以

![打开磁盘管理进行分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414195947.png)

在 C 盘右键选择压缩卷，然后输入需要压缩的大小，就是想要给 C 盘保留多少。我觉得 20G 给 C 盘差不多了。这里直接默认选了，刚刚好：

![选择压缩卷](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414200040.png)

![数据盘和系统盘对半分](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414200448.png)

看到有个属于未分配，我们需要把它变成数据盘。在未分配的盘右键选择新建简单卷，选择之后直接一路默认确定即可。可以修改卷标，我改为 Data, 默认是 “新建卷”。

![新建简单卷](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414200616.png)

![修改卷标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414200834.png)

创建完毕可以在文件管理器查看新建的数据盘。

![文件管理器查看数据盘](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414200943.png)

完成所有的操作之后关键，一定要从菜单关机，而不是直接把虚拟机的窗口关掉。因为如果直接关闭虚拟机窗口，相当于你的电脑直接拔掉电源，有些数据可能会损坏，所以下次启动的时候电脑会启动修复，浪费时间。

![关机](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414201054.png)

# 后续设置

## 备份虚拟机以防万一

虚拟机安装完了，我们可以给它弄个备份，比如下次我们搞老鼠搞鸟把系统弄坏了，重新安装又太慢，我们可以直接恢复原始状态，简而言之就是给它来个快照 snapshot。

打开控制台，进入 windows7.qcow2 所在的位置，输入

```bash
$ ls
setup.sh  win7.desktop  win7.png  windows7.qcow2
$ qemu-img snapshot -c DoneInstall windows7.qcow2
$ qemu-img snapshot -l windows7.qcow2
Snapshot list:
ID        TAG               VM SIZE                DATE     VM CLOCK     ICOUNT
1         DoneInstall           0 B 2023-04-14 20:16:19 00:00:00.000          0
```

`-c` 是创建（create）快照，快照叫做 DoneInstall。`-l` 是列出（list）快照。我们只给电脑来了一个快照，所以这里只有一个快照。下次要恢复就使用

```bash
$ qemu-img snapshot -a ID windows7.qcow2
```

其中 ID 是上面的数字，例如 1 。更多操作可以通过 `man qemu-img` 查看。

## 完善脚本

我们的脚本是从终端启动的，也就是必须开着一个终端。这样太麻烦了，我们把脚本倒数第二行改成

```bash
{
  /bin/taskset -c 9-11 /bin/qemu-system-x86_64 ${options}
}&
```

这样就把虚拟机放到了终端之外运行了。下次要启动虚拟机只需要打开虚拟机目录，然后输入 `./setup.sh` 就可以关闭命令行了。

同时注释掉以下行：

```bash
#options="${options} -boot order=dc"
#options="${options} -drive file=/home/chunshuyumao/Downloads/QEMU/ISOS/w7-64bit.iso,index=1,media=cdrom"
#options="${options} -drive file=/home/chunshuyumao/Downloads/QEMU/VirtIO/virtio-win-0.1.225.iso,index=2,media=cdrom"
```

因为已经安装完成了，这些东西可以不要了，删掉或者直接像上面一样注释掉都可以。


## 设置桌面图标

安装完成并配置好后，我们可以给虚拟机一个图标，这样直接从启动菜单启动虚拟机，而不是命令行，比如下面的效果：

![Windows7 虚拟机的启动菜单图标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414172242.png)

首先下载一个 Windows7 的图片，用于做标识。我选择了 PNG 格式的图片，这个网上随便搜搜即可:

![Windows7 图片，用于做图标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414172415.png)

编写图标文件：

```bash
$ ls
setup.sh  win7.desktop  win7.png  windows7.qcow2
$ vim win7.desktop
$ cat win7.desktop
[Desktop Entry]
Name=Windows 7
Comment=Windows 7 virtual machine
Icon=/home/chunshuyumao/Downloads/QEMU/Win7/win7.png
Exec=/bin/bash /home/chunshuyumao/Downloads/QEMU/Win7/setup.sh
Terminal=false
Type=Application
```

![编写图标文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414172743.png)

写好之后，把文件移动到 

`/home/chunshuyumao/.local/share/applications`

之下。

```bash
$ mkdir -pv ~/.local/share/applications
$ mv win7.desktop ~/.local/share/applications
```

打完收工！这样，GNOME 用户就可以直接从菜单中启动了。

## 安装国产毒瘤

安装是一个简单的事情，这里就不进行演示了，不过还是说说怎么使用共享目录。首先，使用宿主机下载虚拟机需要用的安装包后打开虚拟机，从虚拟机里打开共享目录，然后把安装包复制到虚拟机的其他位置，例如数据盘，然后再安装。

为什么这么麻烦？因为 Windows 系统为了安装起见是不会让你强行运行不明不白的文件的。

安装完软件之后，可以修改下载目录到共享目录，例如：

![阿里云盘的下载位置改为共享目录](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230414203148.png)

# 后语

为了虚拟机的性能，我们可以给宿主机来一个 hugepages —— 大内存页，然后提供给虚拟机。不过这篇文章太长了，没必要再添加了，下次再做一篇新的也可。
