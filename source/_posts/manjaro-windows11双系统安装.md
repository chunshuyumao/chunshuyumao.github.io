---
title: Manjaro + Windows11双系统安装
tags:
  - 双系统
categories:
  - Linux
math: false
mermaid: false
date: 2022-03-04 19:32:58
index_img:
excerpt: 安装双系统是一件简单的事。
---

# 为什么要用 Linux 系统

实际上，使用哪一个系统是没有理由的，个人电脑上基本没有能和  Windows 过招的系统。然而对我来说，  Windows 太麻烦了：8G 内存跑  Windows 还只是勉强流畅，特别是当我微信和QQ一起打开的时候，内存占用到我都怀疑人生了。我确实尝试过重装系统，但是对我来说怎么重装，最后还是会走到讨厌它的那一步。

Windows11 出来之后，  Windows 系统给我的印象就好了很多，特别是它的搜索功能，虽然它早就存在了，毕竟我不喜欢在桌面上放图标。使用它一阵子之后，我还是萌生了换系统的想法。可能这种想法一直存在脑海里，我才不喜欢  Windows 系统。

既然要换系统，那我得找到一个令我满意的系统。我把目光放到了 Linux ，毕竟 MacOS 可望不可及。然而问题是，开源而且免费的 Linux 系统还是有很多分支，多到一只手根本数不过来。此外，这些系统虽然都叫做 Linux ，但是彼此之间差别还是不小。

我最开始接触的 Linux 是 CentOS 。所以当时打算换到 CentOS8 ，虽然它似乎更合适做服务器。然而天有不测风云，谁知道 CentOS Linux 不再更新了。虽然 CentOS Stream 仍然被维护，但是我觉得 CentOS 已经等于被宣判了死刑，因此只能再找出路。

在虚拟机里安装了各种版本的 Linux 之后，我对 Manjaro Linux 和 Arch Linux 有非常大的好感。 Arch Linux 自由度非常高，可定制性强，官方 wiki 也很丰富，是探索 Linux 极好的选择。我花了好一阵子重温命令行安装 Linux 的知识，还给它安装了各种桌面。然而 Arch 总给我一种怪怪的感觉，可能对我来说，它吸引人的方面是拿来把玩，而不是作为主力系统。

但是我很喜欢 Arch ，于是各种搜罗有关 Arch 系统的信息，最后发现了 Manjaro 。 Manjaro 是 Arch 的衍生版，但是和 Arch 又有一定的区别。例如同样作为滚动发行的 Linux ， Manjaro官方包 相比而言比 Arch 更安全一点——因为它不是直接发布的，而是要经过一定的测试。 Manjaro 的另一特点是基本做到开箱即用，而 Arch 本身需要的配置很繁琐——这也是它吸引人的地方。

Manjaro 满足了我懒得折腾的心愿，同时作为 Arch 的衍生品，它与 Arch 共享很多的包。众所周知， Arch 拥有非常强大的社区支持。

Manjaro 官网 [^1] 提供了三种版本的 Manjaro 桌面：
 
1.  XFCE ，轻量级桌面。
2.  KDE ，重量级桌面，实际上现在和 GNOME 也差不多，可定制性高，桌面漂亮。
3.  GNOME ，中量级桌面，简约高效。

![Manjaro官网](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042022699.gif)

我选择了 GNOME ，因为它给我的感觉正像 Arch ，非常简单，没有多余的软件，没有负担。实际上 KDE 也为人称道，特别是它的高定制性，几乎可以定制所有的细节。然而我还是选择 GNOME ，我要的就是简单。既然要简单，自然选择<ruby>最小安装<rt>Minimal</rt></ruby>， LTS 是长期支持版，我没有选择。

![最小安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042028792.png)

---

# 制作启动盘

下载 Manjaro 镜像之后，我们需要制作启动盘，这里推荐使用国人的 Ventoy [^2]。这个小工具看起来小，但是可以引导多个系统。只需要把镜像放到它制作的启动盘里就可以引导多个系统。这有什么好处？通常情况下，我们制作启动盘的方式是格式化U盘，然后使用启动盘制作软件制作启动盘(格式化是必须的，有时是自己手动，有时是启动盘软件格式化)。这样，一个U盘无论多大，一次只能引导一个系统，要引导新系统时都需要格式化。一来麻烦，二来对U盘的使用寿命也不好。当然，最重要的是， Ventoy 引导多个系统的能力听起来就很厉害。

有一个教训是我直接给自己的电脑装了 Manjaro 之后，由于某些需要想要装回  Windows ，此时所有的启动盘制作软件制作的启动盘都没有用。制作启动盘之前必须格式化U盘，所以当我把系统搞坏之后连原本的 Linux 系统也装不了。最后只能手动制作启动盘，纯手工给U盘分区，因为我要安装的系统超过了 fat32 文件系统的最大上限。

因此使用 Ventoy 是一项保险的举措，至少等你后悔的时候，它不会让你失望。

## 下载Ventoy

我们可以到官网下载 Ventoy 。如果不想安装，可以下载 便携版 ，开箱即用。

![Ventoy官网](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042039980.png)

## 开始制作

制作启动盘很简单，打开 Ventoy ，然后选择U盘即可。通常情况下， Ventoy 会自动识别你的U盘。

![启动Ventoy](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042050020.png)

![制作启动盘](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042051376.png)

安装完之后，把你的 ISO 镜像复制到 U盘。你会发现 U盘 仍旧是空的，实际上只是你看不见其他文件而已。

![复制ISO镜像](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042057885.png)

---

# 电脑分区

对我来说， Linux 够用了，但是谁也不知道可能又需要  Windows 专有的软件。既然要安装双系统，而且是 Linux 作为主系统，那只需要给  Windows 保留一小部分的空间。

  Windows 分区比较简单，系统就自带了。我们只需要搜索“分区”就会弹出需要的软件，选择就好了。搜索使用 <kbd> Windows</kbd>+ <kbd>s</kbd>。

![搜索分区软件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042058958.png)

![查看硬盘](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042059442.png)

分区的时候，固态硬盘(C盘)分出一部分，待会安装 Manjaro ；机械硬盘也要分，主要作为存储区。我是在已经安装完毕的电脑演示的，所以可以看到，实际上我的电脑已经分成了很多的区。

要分区，先压缩一定的空间，通常是你想要分配多少就要所多少。记住， 1G=1024M 。

![演示分机械硬盘](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042102453.png)



![压缩](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042104626.png)

压缩完之后新建 卷 。

![压缩完之后](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042104738.png)

![继续选择](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042106771.png)

![分配驱动号](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042106229.png)

新建卷标，最好是有意义的卷标。

![设置卷标，建议设成有意义的卷标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042107720.png)

分区完成。按照这个方式给固态硬盘分区。

![分区完成](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042108967.png)

![成果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042109229.png)

至此，我们已经分区完成了。注意，后面我们要做的就是把 Linux 安装到我们分出來的 新盘 。

---

# 进入BIOS或者UEFI引导界面

 BIOS 和 UEFI 是系统安装的引导界面，也就是我们给老电脑开机的时候，看到的黑乎乎的界面。每个厂商的引导界面的快捷键都不一样，最常见的是 <kbd>F2</kbd> 和 <kbd>F12</kbd>。例如我的是华硕电脑，使用 <kbd>F2</kbd>。通常这些东西都是可以在网上查到的。

那什么时候按呢？简单，就是重启电脑的，等到刚刚出现厂商标志的时候，以迅雷不及掩耳之势按住你电脑的按键。当然，最傻的方式是电脑重启黑屏之后就一直按你的快捷键。

进入 BIOS 或者 UEFI 之后，先去高级模式( Advance Mode )把 Boot 的 Security 改成 disable(关闭) 。然后选择 Boot Mode 中的U盘启动。当然，如果你的界面和我的一样，你也可以选择调整右手边的 Boot Priority ，把U盘调到最前面。

![选择BootMode](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042120262.jpg)



![选择U盘启动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042125263.jpg)

选择想要安装的系统。我这里有两个系统，一个 Arch ,一个 Manjaro ，选择 Manjaro ，回车。

![Ventoy引导界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042126910.jpg)

这里可以进行简单的配置。然后选择 open source drivers 。 open source 是开源，通常等于使用 Intel 显卡； proprietary drivers 就是闭源，自然就是 Nvidia 了。我的电脑是双显卡，所以使用哪一个都差不多。不过说实话，我使用闭源的 Nvidia 的时候出现了比较大的问题：开机容易卡在黑黢黢的界面。修改之后直接回车。实际上也不用修改，后面还是可改的。

![这里简单配置一下](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042132829.png)

如果你的电脑出现一堆的文字，那是正常现象，不用担心。

![启动](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042137953.png)

等到进入这个界面就说明没问题了。

![Live 界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042139339.png)

这里可以体验一下 Manjaro ，因为所做的修改都不会保留下来。

---

# 安装

记得连接网络，点击 右上角 ，连接网络。

![记得连接WIFI](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042141928.png)

连接网络之后开始安装，点击 右下角 的安装软件——最右下角，开始安装。

选择语言等基本配置。

![开始安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042145530.png)

一直回车到分区界面，选择 手动分区 (manual partitioning)。

![选择手动分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042146756.jpg)

选择磁盘，找到我们之前的分区，通过 标签 (卷标)可以查看我们的分区——所以分区的时候一定要给一个有意义的卷标。

![(选择我们的分区)](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042150337.jpg)

点击我们分出的区域——注意，一定要选择我们的分区，例如我的分区 Linux ，然后选择 创建(Create) ，给大小 300M 就可以了， 文件系统 (file system)是 fat32 ，挂载点是`/boot/efi`，给一个卷标，标识为`boot`，选择`OK`。

![分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042157081.png)

选择 空闲空间 (free space)，继续新建。这里有两种情况：

## 你有两块硬盘，一块固态硬盘，一块机械硬盘，并且分别分出了两个分区给Linux

分区布局：

1. EFI 分区，固态硬盘，300M足矣。
2. /分区(root)，固态硬盘，剩余的空间。
3. swap 分区，机械硬盘，和自己的内存一样大。
4. /home 分区，机械硬盘剩下的三分之二以上。
5. /opt 分区，机械硬盘最多三分之一。

固态硬盘剩余的所有空间新建一个分区，作为 根 (root)挂载点，文件系统是`ext4`。

![根分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042203158.png)

然后点击顶部的存储器，选择机械硬盘。同理选择自己分出來的那一部分——说过了，看自己的卷标识别，千万不要在固态硬盘和机械硬盘上使用 Windows系统的分区，不然没人救得了你。

分出一部分的空间，新建一个 swap 分区， swap 大小是你的内存大小。我的是 8G 内存，所以选择 8G 。文件系统选择`linuxswap`，标记`swap`，卷标自定义，最好有意义。不用选择挂载点。

![swap分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042211258.png)

最后，把你机械硬盘的剩余空间分成两部分——四六或者三七分，少的部分作为`/opt`挂载点，文件系统选择`ext4`，没有标记：

![opt分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042215243.png)

最后剩余的空间放到`/home`分区，文件系统还是`ext4`，无标签，卷标可以给个名字

![home分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042217101.png)

至此，你的分区差不多了。最后你的分区看起来像这样(注意，原本还有 windrows 系统的分区，这里演示不了。下面是固态硬盘分区和机械硬盘的分区)：

![固态分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042220498.png)

![机械硬盘分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042222826.png)

## 你只有一块分区

参照两块硬盘的分区，只不过不用分出`/opt` 和`/home`分区而已。

分区布局：

1. EFI 分区，同上。
2. swap 分区，同上。
3. /分区(root)，剩余的所有空间。

![分区如下](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042229457.png)

分区一直是一个见仁见智的话题，如果你有自己的想法，可以不参照我的方案。

---

# 安装

最后填写一些信息，然后就安装，重启。

![配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042226386.png)

![安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203042227959.png)

至此， Manjaro 安装完毕。

---

# 后语

安装完毕之后，我们还需要安装一些常用的软件、使用中文拼音等，但是一篇写不完，所以后面我会再详细介绍怎么安装常用软件以及美化自己的系统。

---

# 相关网址

[^1]: https://manjaro.org/
[^2]: https://www.ventoy.net
