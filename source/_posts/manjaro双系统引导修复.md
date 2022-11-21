---
title: Manjaro 双系统引导修复
tags:
  - 引导修复
categories:
  - Linux
math: false
mermaid: false
date: 2022-03-05 16:40:30
index_img:
excerpt: 人生苦短，何必折腾？
---

# 前言

人在江湖飘，哪有不挨刀。我装双系统之前，电脑只安装了 Manjaro ，后来以防万一装了 windows 。因为引导  Windows11 一直失败，所以我先安装了  Windows10 。后来把所有的问题都搞定之后，我就给我的  Windows10 升级。升到  Windows11 本来是好事，结果启动 Manjaro 的时候，直接进入了 grub rescue 模式。

在此之前，我完全不相信双系统会出现这种问题。毕竟我给我的硬盘分好了区，两个系统井水不犯河水。当时觉得别人出现问题是因为他们没有正确处理这些问题。当我的 linux 系统的 grub 被 Windows 弄坏的时候，我知道我也中招了。

这种问题按理来说不是难事，可是当我跑到网上去查的时候，发现能查出来的都是不知所云的。果然百度不能信。求天求地不如求己，因此又有了这篇引导修复的文章。

---

# 引导修复

我的 Manjaro 系统出现故障的时候，只要一选择 Manjaro 启动就会跑到 grub rescue 界面，所以如果你可以进入正常的引导界面，只是启动后黑屏的话，多半是驱动问题，这篇文章自然就不用看了。

引导修复有两种办法：

1. Live CD修复
2. 手动修复

第一种方法对于 Windows 玩机大家来说很简单了，就是各类的 PE机 做的事。不过我还是说说。

## Live CD修复

这种方式要求你有一个 Manjaro 启动盘或者和你系统一样的启动盘。为什么使用 Manjaro Live CD ？因为 Manjaro 本身支持 LiveCD 。启动盘不难制作，按我之前说，直接用 Ventoy [^1]制作即可。

启动电脑进入 BIOS 或者 UEFI 界面，选择U盘启动，就像安装 Manjaro 一样直到进入图形界面。

进入图形界面之后，打开控制台，输入

```shell
sudo fdisk -l
```

查看你的硬盘分区，通常你是可以知道自己的硬盘分区的。例如分区如下，我的 Manjaro 安装在 Windows 之后，所以倒数一二块分区应该就是我的 Manjaro 的 EFI 和 根(root) 分区:

```shell
Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: INTEL SSDPEKNW512G8                     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2CBB82D2-DA15-824E-A9B0-AF96C6143278

Device             Start        End   Sectors   Size Type
/dev/nvme0n1p1      2048     616447    614400   300M EFI System
/dev/nvme0n1p2    616448     649215     32768    16M Microsoft reserved
/dev/nvme0n1p3    649216  159879167 159229952  75.9G Microsoft basic data
/dev/nvme0n1p4 159879168  161349631   1470464   718M Windows recovery environmen
/dev/nvme0n1p5 161351680  161966079    614400   300M EFI System
/dev/nvme0n1p6 161966080 1000215182 838249103 399.7G Linux filesystem
```

接下来是挂载我的 Manjaro 两个分区，同时转换到挂载的目录：

```shell
sudo mount /dev/nvme0n1p6 /mnt
sudo mount /dev/nvme0n1p5 /mnt/boot/efi
manjaro-chroot /mnt
```

转换之后，命令提示符应该从原本的 Manjaro 格式变成其他格式，我的变成了这样

```shell
sh-5.1#
```

这时候，我们需要更新我们的 grub ，然后再次安装它：

```shell
update-grub
grub-install /dev/nvme0n1p5
```

当然，你要是不放心，还可以再次生成 grub ：

```
grub-mkconfig -o /boot/grub/grub.cfg
```

然后退出，卸载，然后重启

```shell
exit
sudo umount -R /mnt
reboot
```

正常情况下，你的系统应该可以启动了。

## 手动修复

这种情况是你手中没有 Live CD ，或者没有合适的 Live CD 。这种就很难受了。

首先你会看到这个提示

```shell
eorror: unkown filesystem
Entering rescue mode...
grub rescue > 
```

如果没有这个提示，很好，你的问题我不能解决，你可以另往它处了。

第一步，我们先看一下自己的 Manjaro 系统在哪里，使用`ls`。

```shell
ls
```

我的提示如下

```shell
(hd0) (hd1) (hd1,gpt4) (hd1,gpt3) (hd1,gpt2) (hd1,gpt1) (hd2,gpt6) (hd2,gpt5) (hd2,gpt4) (hd2,gpt3) (hd2,gpt2) (hd2,gpt1) 
```

你的分区可能不会这么多，所以不用担心。

接下来就是烦人的操作了，使用`ls`一个个查看这些分区，例如

```shell
ls (hd0)/
ls (hd1,gpt4)/
...
```

注意，hd1和gpt4之间除了逗号，什么也没有，括号之后还需要加上`/`，除此之外不要添加别的东西。

一个一个地试，你可以会得到这类消息

```shell
error: unkown filesystem
```

恭喜你，你成功排除了一个选项，请继续排除。

等到你找到类似下面这个

```shell
ls (hd2,gpt6)
./ ../ lost+found/ boot/ dev/ home/ opt/ proc/ run/ sys/
```

的提示的时候，这说明你找对了，只要看到`boot/`就说明已经正确了。记住你的卷标，我的是`(hd2,gpt6)`

接下來输入

```shell
set root=(hd2,gpt6)
set prefix=(hd2,gpt6)/boot/grub
insmod normal
normal
```

至此，你的引导系统算是正确了。它会自己重启。

不急，等你进入系统之后打开控制台(终端),输入

```shell
fdisk -l
```

查找你的 Manjaro 的 EFI 在哪里，我的是`/dev/nvme0n1p5`，我知道，上面我就试过了。然后输入

```shell
sudo update-grub
sudo grub-install /dev/nvme0n1p5
```

重启一下，你的系统又变好！！

---

# 后语

只要不乱搞，你的系统是不会有问题的。所以当你的系统有问题的时候，别急着责怪电脑，毕竟你比电脑还不负责。

---

# 相关网址

[^1]: https://ventoy.net/
