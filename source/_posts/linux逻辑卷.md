---
title: linux逻辑卷
hide: true
tags:
  - 逻辑卷
  - LVM
categories:
  - Linux
math: false
mermaid: false
date: 2022-03-14 21:49:28
index_img:
excerpt: 讲讲逻辑卷
---

# 1. 逻辑卷(Logical Volumn Memory)

#todo

# 2. 创建逻辑卷

使用 `fdisk -l` 查看自己想要硬盘

我的虚拟机准备了 4 块硬盘，每一块都是 8 G —— 硬盘空间没有要求一定相同。

![准备的硬盘](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203142154199.png)

首先创建<ruby>物理卷<rt>Physical Volume</rt></ruby>

```shell
pvcreate /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

> 上述命令也可以写成 `pvcreate /dev/sd{b,c,d,e}`
>
> 如果你的 Linux 提示没有 pvcreate 命令的话，先下载 lvm2 软件。Arch 系 Linux 使用 `pacman -S lvm2` 进行安装。

不出意外的话，你的创建<ruby>物理卷<rt>Physical Volume</rt></ruby>之后会提示

```shell
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.
```

使用 `pvs` 进行查看是否创建成功。

```shell
# pvs
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb      lvm2 ---  8.00g 8.00g
  /dev/sdc      lvm2 ---  8.00g 8.00g
  /dev/sdd      lvm2 ---  8.00g 8.00g
  /dev/sde      lvm2 ---  8.00g 8.00g
```

<ruby>物理卷<rt>Physical Volume</rt></ruby>是不能直接使用的，我们需要在其上再抽象一层<ruby>卷组<rt>Volume Group</rt></ruby>。

```shell
vgcreate extraVG /dev/sdb /dev/sdc
```

其中 `extraVG` 是新的卷组名，随便给它一个名字；`/dev/sdb`等，是想要用于创建卷组的硬盘。这是再次查看<ruby>物理卷<rt>Physical Volume</rt></ruby>就可以看到卷组了

```shell
  # pvs
    PV         VG      Fmt  Attr PSize  PFree 
    /dev/sdb   extraVG lvm2 a--  <8.00g <8.00g
    /dev/sdc   extraVG lvm2 a--  <8.00g <8.00g
    /dev/sdd           lvm2 ---   8.00g  8.00g
    /dev/sde           lvm2 ---   8.00g  8.00g
```

使用 `vgs` 查看<ruby>卷组<rt>Volume Group</rt></ruby>：

```shell
# vgs
  VG      #PV #LV #SN Attr   VSize  VFree 
  extraVG   2   0   0 wz--n- 15.99g 15.99g
```

可以看到，使用 `vgs`显示 `extraVG` 有两个<ruby>物理卷<rt>Physical Volume</rt></ruby>，没有<ruby>逻辑卷<rt>Logical Volume</rt></ruby>。现在我们要创建一个<ruby>逻辑卷<rt>Logical Volume</rt></ruby>。

```shell
lvcreate -L 12G -n extraLV extraVG
```

> `-L` 表示<ruby>逻辑卷<rt>Logical Volume</rt></ruby>大小，
> `-n` 表示<ruby>逻辑卷<rt>Logical Volume</rt></ruby>名字，
> 最后一个参数是使用哪个<ruby>物理卷<rt>Physical Volume</rt></ruby>创建<ruby>逻辑卷<rt>Logical Volume</rt></ruby>

使用 `lvs` 查看<ruby>逻辑卷<rt>Logical Volume</rt></ruby>。

# 3. 挂载逻辑卷

创建一个目录 `mkdir /mnt/extraDir`，然后挂载

