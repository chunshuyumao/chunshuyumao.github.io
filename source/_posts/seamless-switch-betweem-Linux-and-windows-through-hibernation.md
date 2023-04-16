---
title: 通过休眠在 Linux 和 windows 之间无缝切换
hide: false
tags:
  - Hibernation
categories:
  - Linux
math: false
mermaid: false
date: 2023-04-15 14:57:39
---

# 前言

Linux 电脑挂了一个 Windows 虚拟机，但是有些东西还得是真机才管用，比如 Windows 大型独占软件，或者备份国内某空间占用贼大但只要你一找文件它就告诉你文件已过期的社交软件的聊天记录。备份可以在虚拟机中完成，但是只能是电脑连接手机热点，速度太慢了咱没那时间。

很多时候自己可能刚在 Linux 上工作或者刚在 Windows 上玩游戏，但是突然接到任务要在 Windows 上运行某个东西或者要换到 Linux 上办公，我不希望关掉 Linux 或者 Windows 系统，因为任务并不复杂，很快就弄好了。此外，自己的电脑布局可能很多，开机回来再一个一个打开可能太慢了。

这时可以选择休眠，休眠会保存系统的运行状态，直到下次开机它才帮你恢复。这也就是本篇文章的主要内容：在 Windows 和 Linux 系统设置休眠，保存系统状态。

# 设置休眠

电脑系统存在多种挂起的方案，其中比较重要的是：

1. Suspend to RAM, suspend. 挂起，一般就是电脑显示的挂起，它把状态保存到 RAM（random access memory），也就是内存当中。这个状态就是离开你电脑不久它自动进入的熄屏状态。
2. Suspend to disk, hibernate. 休眠，一般休眠之后会关机，它把状态保存到硬盘，实际上是交换空间（swap space），也就是交换分区（swap partition）。这个状态一般不会自动进行，而且由于把状态保存到交换分区，它可以实现电脑开机后恢复关机前状态。

## Linux 休眠

在安装 Linux 的时候会有一个分区过程，一般分根分区（root）、家分区（home）和交换分区（swap）三个分区，根分区占十分之三，交换分区是电脑内存的一倍到两倍，剩下的给家分区。这里说的电脑内存是 RAM, 一般叫做运存 —— 手机上这么叫。比如我的电脑内存是 8G，那交换分区一般可以给 8G 或者 16G。

很多人会说，现在什么年代了还用交换分区？交换分区确实刚开始是为了解决内存太小出现的，但是人家也可以用来做其他的事，比如现在要说的休眠。一般来说，交换分区应该是内存的两倍，这样当休眠时，交换分区刚好可以保存运行状态的所有的数据，而且还有盈余，所谓宁缺勿滥。

不过我倒霉了，我后来给自己的电脑加了一个内存条，内存变成了 16G, 而我的交换分区也只有 8G。不过不用担心，一般来说问题不大，因为还是有机会的。我一直正常使用也没出现问题。

### 配置根文件系统

Linux 启动涉及到比较复杂的过程，其中大部分需要使用到一个比较小的、完整的系统，叫做 initramfs(initial RAM filesystem)，根文件系统。如果我们需要在开机的时候恢复上次关机前电脑的状态，那就需要在这里给根文件系统任务。

首先打开 `/etc/mkinitcpio.conf`，找到 HOOKS 一行，添加 `resume` 钩子：

```sh
$ vim /etc/mkinitcpio.conf
...
#HOOKS=(base udev autodetect modconf kms keyboard keymap block filesystems fsck) 
HOOKS=(base udev autodetect modconf kms keyboard keymap block filesystems resume fsck)
...
```

我上面把原来的钩子注释点，然后再添加一行，这样避免出错无法还原。 `resume` 需要在 `udev` 后面，因为这些钩子是按顺序调用的，交换分区需要 `udev` 检查，如果在调用 `udev` 之前使用 `resume`，那系统就找不到交换分区了，得先让 `udev` 把交换分区找出来。

我在 `resume` 当在 `fsck` （文件检查）之前，`udev` 之后，不知道有什么深意，俺忘了。不过不重要，只需要记住在 `udev` 之后即可。

写好的之后需要重新生成 initramfs，即在命令行上调用 `sudo mkinitcpio -P`。命令会诱发很多的输出，不过这不重要。

```sh
$ sudo mkinitcpio -P
```

### 设置内核参数

`initramfs` 配置的是钩子，也就是 `resume` 后需要处理的东西，我们需要在内核参数（kernel parameters）上告诉系统从哪里恢复上次的状态。

首先确认自己的交换分区是哪个分区，一个简单的办法是使用 `sudo fdisk -l`：

```sh
$ sudo fdisk -l
...
Device         Start        End    Sectors   Size Type
/dev/sda1       2048  275800063  275798016 131.5G Microsoft basic data
/dev/sda2  292577280  502292479  209715200   100G Linux filesystem
/dev/sda3  502292480 1953525134 1451232655   692G Linux filesystem
/dev/sda4  275800064  292577279   16777216     8G Linux swap
...
```

上面可以看到，有个 Type 写的是 Linux Swap, 这就是交换分区，是 `/dev/sda4` 分区。

也可以使用 `cat /etc/fstab` 查看：

```sh
$ cat /etc/fstab
# <file system> <dir> <type> <options> <dump> <pass>
...
# /dev/sda4 LABEL=ArchLinuxSwap
UUID=92b7f976-98c5-4eef-9fc7-ba037fd8552e       none            swap            defaults        0 0
...
```

也可以看到注释上写的 `/dev/sda4` 就是交换分区。

用编辑器打开 `/etc/default/grub`，找到 GRUB_CMDLINE_LINUX 一行，如果没有就自己添加一行，在这行的末尾引号内添加 `resume=你的交换分区`。我的是 `/dev/sda4` 所以写 `resume=/dev/sda4` ，也可以用 UUID 替代，也就是 `resume=UUID=92b7f976-98c5-4eef-9fc7-ba037fd8552e`。

```sh
$ vim /etc/default/grub
...
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
GRUB_CMDLINE_LINUX="nvidia_drm.modeset=1 resume=/dev/sda4"
...
```

如此保存退出即可。保存之后需要重新生成 `grub` 文件，也就是在命令行输入:

```sh
$ sudo grub-mkconfig -o /boot/grub/grub.cfg
```

同样也有一堆输出。

### 尝试 Linux 休眠

弄好之后，Linux 休眠可以通过 `systemctl` 来启动。在命令行上输入：

```sh
$ sudo systemctl hibernate
```

然后电脑就会进行休眠，等到屏幕和键盘灯都灭了就可以开机进入 Windows 进行休眠设置了。

## Windows 系统休眠

关机后启动 Windows 系统，打开搜索，搜索电源选项，然后选择“选择电源按钮的功能”，将电源按钮的功能改为“休眠”。下面是 Windows7 的演示：

![搜索电源选项](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023043/20230415155931.png)

![将电源按钮的功能改为“休眠”](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023043/20230415160008.png)

Windows10 和 Windows11 也是一样，只是可以设置的可能更多，例如接通电源如何、使用电池如何、睡眠按钮如何、关闭盖子如何，只要按自己的想法就可以了。

Windows 配置好之后，不要关闭控制面板，留着直接按电源键进行休眠。休眠后重新启动进入 Linux，看看是不是保留着原本的工作状态，如果是，可以再休眠进入 Windows 看自己的控制面板还在不在。

对于 Windows，把电源键改成了休眠该怎么真的关机？可以直接使用快捷键 `alt+f4` ，然后选择关机或者重启即可。


# 后语

一篇文章控制在 2000 字左右差不多了，后面的计划是写在 Linux 上使用 Nvidia 显卡。GNOME 桌面对 Nvidia 显卡的支持更好一点，而且我也没在 KDE 上试过 —— 我更喜欢 GNOME 桌面，所以后面的设置主要针对的是 GNOME 的显示管理器 GDM（GNOME Display Manager）。
