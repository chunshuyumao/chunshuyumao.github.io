---
title: Linux 中开启大内存页
hide: false
tags:
  - Hugepages
categories:
  - Linux
math: false
mermaid: false
date: 2023-04-15 08:26:28
---

# 前言

[上篇文章](https://chunshuyumao.github.io/2023/04/14/using-windows7-as-guests-of-QEMU/) 说到，使用 QEMU/KVM 可以开启大内存页（Hugepages），以获得更好的虚拟机性能。不过当时篇幅太长没有继续写下去，今天打算补一下。

本篇文章基本参照 [Arch KVM Wiki](https://wiki.archlinux.org/title/KVM#Enabling_huge_pages "Arch KVM") 的步骤进行配置，说句不好听的，可能只是翻译成了中文，如果有能力可以直接看原文的请按提供的网址查看。

# 启用大内存页

大内存页允许操作系统提供更大的内存申请，一般大内存页大小在 2M 以上，默认的内存页是 4K。[大内存页在 Linux2.6 之后就默认编入内核，只要内核不太老，一般没啥需要担心的](https://docs.oracle.com/database/121/UNXAR/appi_vlm.htm#UNXAR391 "Hugepages in Kernel")。

## 什么时候用大内存页

进程向操作系统申请内存不是用多少申请多少而是有一个申请原子值，也就是最小的申请值。比如有一个 0.9K 大小的文件，不是说它在内存中就占用 0.9K，而是系统直接给它分配默认值大小的空间，一般是 4K。因此很多时候，0.9K 和 3K 占用的空间是一样的。现在的操作系统在使用上肯定有所优化，但是基本道理如此。

进程对内存操作的时候也会返回一部分内存给操作系统。假设有一个进程占了 4K，下一个进程就要从 4K 开始使用内存，第一个进程又申请了 4K。此时如果第二个进程将内存返还给了操作系统，系统的内存不会自动对齐，而是保留原来的状态，即第一个 4K 和第三个 4K 被第一个进程使用，第二个 4K 则为空闲。这就是内存不连续。

第一个进程怎么知道自己使用的内存在哪里？这就涉及到寻址，为了找到自己使用的空间，第一个进程需要慢慢找到。怎么找？按照一个字节一个字节找也可以，但是太慢了。这时人为规定，内存页以 4K 对齐，按照 4K 找！然后进程只需要走两步就找到了自己使用的第二块内存，这比一个一个字节快很多。这里的 4K 是人为规定的，是经验值。

如果进程需要进行大规模的内存申请和释放，例如现在的人工智能学习和虚拟机运行，就会造成大量的内存不连续。内存不连续寻址会变慢，而且由于内存频繁申请和释放，操作系统也会更加繁忙。

为了应对这种内存频繁申请和释放的情况，一个比较好的方法是，一开始就划分一个大内存，叫做内存池，比如 16K, 直接分配给进程，这样进程的内存就连续了，也不会花太多时间进行寻址。那上面的例子来说，第一个进程第一次只需要 4K, 我们给了 16K。第二个进程需要 4K, 我们也给另外的 16K。等到第一个进程需要另外的 4K 时就不需要分配内存了，因为我们给它的够多了，它可以继续在自己的 16K 里使用另外所需的 4K。这样第二个进程申请和释放内存和第一个关系不大，没啥影响也就避免了内存断裂和频繁将计算机算力用在寻址上。

这样虽好，但是我们也看到，第一个进程只用到 8K，第二个 4K，合起来也就是 12K，但是我们总共分配出去 32K，很多内存根本用不到！因此大内存页一般只用于对内存操作频繁的地方，这正是虚拟机的用武之地。

## 检查大内存页是否已经启用

在启用之前，我们需要确认自己的电脑是否已经启用了大内存页。一般是没有的，但过程还是要走。打开控制台输入 `file /dev/hugepages`。如果出现

```sh
$ file /dev/hugepages
/dev/hugepages: sticky, directory
```

则表明你的 Linux 系统已经启用了大内存页，后面不必折腾了，直接跳到[分配内存页个数](#分配内存大小)

如果出现

```sh
$ file /dev/hugepages
/dev/hugepages: cannot open `/dev/hugepages' (No such file or directory)
```

那就说明没有启用，我们需要创建这个目录，输入

```sh
$ sudo mkdir -v /dev/hugepages
mkdir: created directory '/dev/hugepages'
$ ls -l /dev
drwxr-xr-x   2 root         root           40 Apr 15 09:08 hugepages
```

创建好目录看到目录权限是第一排的 0755，属于 root 用户和 root 用户组，我们需要修改权限。

用自己喜欢的编辑器打开 `/etc/fstab` ，添加：

```sh
$ vim /etc/fstab
hugetblfs  /dev/hugepages  hugetblfs  mode=01770,gid=kvm  0  0
```

这是让系统开机之后自动帮你挂载这个目录并修改权限，目录属于 kvm 组，`gid` 表示目录所属的组 ID。

查看电脑是否有 kvm 组，输入 `grep kvm /etc/group`:

```sh
$ grep kvm /etc/group
kvm:x:992:qemu,chunshuyumao
```

出现这样的信息说明有 kvm 组，如果没有出现，那就是没有，我们需要新建一个 kvm 组，然后把自己的账号加入其中。

```sh
$ sudo groupadd kvm
$ sudo usermod -aG kvm chunshuyumao
$ groups chunshuyumao
wheel kvm chunshuyumao
```

第一行是添加新的组，第二行是把用户（chunshuyumao）添加（a）到 kvm 组（G），第三行是查看用户（chunshuyumao）所在的组（groups）。我的用户属于三个组，分别是 wheel、kvm 和同名组 chunshuyumao。

## 挂载大内存页

上面设置好后，在命令行上输入：

```sh
$ sudo umount /dev/hugepages
$ sudo mount /dev/hugepages
$ mount | grep hugepages
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,gid=992,mode=1770,pagesize=2M)
```

看到上面的结果就说明没啥问题了。可能输出和我的不大一样，但有输出就说明没问题。

接下来可以看看大内存页的信息，输入

```sh
$ grep -i hugepages /proc/meminfo
AnonHugePages:    731136 kB
ShmemHugePages:        0 kB
FileHugePages:    256000 kB
HugePages_Total:    2050
HugePages_Free:     2050
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

可以看到，内存页大小（Hugepagesize）是 2M，总共有 2050 个内存页，也就是 2 x 2050 / 1024 ⋍ 4G，大内存页总共有 4G。这是当初运行 Windows10 虚拟机的内存大小的两倍。因为电脑已经设置了大内存页个数，所以 `HugePages_Total` 是 2050 个大内存。如果之前电脑没有开启过大内存页，这里应该是 0。

一般来说，给自己的虚拟机几个 G 内存，内存页就应该是其两倍。我给了 4G 是以前虚拟机所需内存的两倍。现在使用 Windows7 可以把内存页缩小到 2G，但是保留着也没啥。

分配内存的时候也不要太死板，不是 2G 就铁定了 1024, 可以稍微扩大一点，例如 1030 取个整，因此我的 4G 原本应该是 2048，但我取了整，选择 2050。

## 分配内存页个数

上面我们挂载了内存页，现在需要给内存页分配大小了。比如我分配 4G，就是 2050 个 2M。在控制台上输入：

```sh
$ echo 2050 | sudo tee /proc/sys/vm/nr_hugepages
```

Linux 上无法使用 `sudo echo`，所以我们用 `tee` 作为中介，这是一个技巧。完成后可以输入

```sh
$ grep hugepages /proc/meminfo
AnonHugePages:    731136 kB
ShmemHugePages:        0 kB
FileHugePages:    256000 kB
HugePages_Total:    2050
HugePages_Free:     2050
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

查看。原本应该是 0 的 `HugePages_Total` 此时应该是 2050 了。

## 在 QEMU 中使用大内存页

打开自己的虚拟机启动脚本 setup.sh，在选项处添加:

```sh
$ vim setup.sh
...
...

options="${options} -mem-path /dev/hugepages"
{
  /bin/taskset -c 9-11 /bin/qemu-system-x86_64 ${options}
}&
unset options
```

打开虚拟机，查看是不是刚好用了那么多内存。

```sh
$ grep -i hugepages /proc/meminfo
AnonHugePages:    757760 kB
ShmemHugePages:        0 kB
FileHugePages:    256000 kB
HugePages_Total:    2050
HugePages_Free:     1542
HugePages_Rsvd:        4
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

可以看到，我的虚拟机开机后使用的 2M 大内存页 2050 - 1542 = 508 个，差不多是 1G 内存。

## 大内存页持久化

直接修改 `/proc/sys/vm/nr_hugepages` 只会影响现在的电脑状态，重新开机后内存页又变成 0。为了让内存页持久化，我们需要让系统开机的时候自己设置内存页个数，也就是所谓的持久化。

用编辑器打开 `/etc/sysctl.d/40-hugepage.conf`，如果没有 40-hugepage.conf 就创建一个:

```sh
$ vim /etc/sysctl.d/40-hugepage.conf
vm.nr_hugepages = 2050
```

# 后语

我发现自己发的博客，科普占了很大的内容，好像真正操作蛮简单的样子。






















