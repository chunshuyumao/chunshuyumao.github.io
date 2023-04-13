---
#title: after_a_year_of_using_linux_as_the_main_computer
title: 使用 Linux 作为主力机一年后
hide: false
tags:
  - Linux
categories:
  - Linux
  - 杂谈
math: false
mermaid: false
date: 2023-04-13 14:06:29
---

# 前言

一年前，当我把自己的电脑系统改成 Linux 时，我就想着一年快过去、快过去，我要写一篇文章讲讲自己用 Linux 系统如何了！然后真到了一年后，我却懒得写了。现在抽出时间，恰好可以圆一下去年的梦。作为一个总结，我会聊聊两个问题，一个说说自己的感受，一个说说使用 Linux 作为主力机需要的 Windows 替代品。

# 使用 Linux 作为主力机感觉如何？

首先的评价，Linux 作为主力机其实和 Windows 是差不多的。很多人总是在比较 Windows 和 Linux 的优缺点，但是很少注意到一点：普通人使用电脑从来不是单单使用系统。论操作系统，BDS 系统可以说是最完美、最优雅的，和 Linux 比没有那么分裂，没有成千上万的发行版动不动就互掐；和 Windows 比则更加优雅，而且还是开源的。但现在个人电脑基本没有几个用 BSD 系，甚至 BSD 系的大哥 FreeBSD 还是 MacOS 的免费小白鼠，靠全世界的资本家豪掷几千美元糊弄日子。所以在内核、系统设计上比较操作系统的优劣是没有任何意义的，否则 FreeBSD 不会活的比使用它内核的 MacOS 还差。大多数人使用电脑用的是电脑软件，也就是系统的生态，这方面 Windows 有绝对的优势。

Windows 对计算机图形化的影响是革命性的，极大的降低了普通人使用电脑的门槛。Linux 则先天不足，因为它继承了老的 Unix 传统，并没有发展出合理、实用、稳定的图形界面。现在的 Linux 系统使用的两大图形界面，[KDE(Kool Desktop Environment)](https://kde.org/ "KDE") 和 [GNOME(The GNU Network Object Model Environment)](https://www.gnome.org/ "GNOME") 基于比 Linux 还要古老的 [X.org](https://www.x.org/wiki/ "X.org")，在设计上可能没有什么大问题，但是在实践中表现确实不好。Linux 中的桌面系统是属于软件行列，是可选择的，如果你不喜欢可以使用其他桌面，所以 Linux 的自由、开源带来的分裂也延续到了桌面系统上。在 KDE 和 GNOME 基础上创造的桌面又不可胜数。为了解决 Linux 桌面的短板，[Wayland](https://wayland.freedesktop.org/ "Wayland") 被创造出来作为 X.org 的替代品，但是纯靠爱发电让 Wayland 经过十几年的发展也没有从 Windows 口中撕下一块肉，反倒是和 X11 争原本就少得可怜的桌面份额，而且表现不是很好。

![KDE 是最早的 Linux 桌面，走 Windows 桌面风格，由于许可证陷入纠纷导致大厂不敢使用，背靠 Qt 公司，实力雄厚，最近几年的 Qt 越做越大，KDE 也逐渐吃香，可能是最具有前景的 Linux 桌面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413152525.png)

![GNOME 是 KDE 陷入官司后被创造出来替代 KDE 的桌面，以简洁、高效自称，桌面风格类似 MacOS，是许多 Linux 系统的默认桌面，包括著名的红帽系统（RedHat Enterprise Linux, RHEL），实际上，红帽是 GNOME 的最大支持者](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413152902.png)

说了这么多，似乎 Linux 就是一点用也没有？其实也不是，我说了，电脑被使用的是生态不是系统本身。最近几年 Linux 系统磅礴发展，特别是国内出了一个[深度系统（Deepin）](https://www.deepin.org/ "Deepin")加深了国人对 Linux 系统的认识。最近几年 Linux 的生态逐渐转好，所以很多国内的企业也开始在 Linux 上抓市场。可能大多数人是受制于某个无形的手，但是他们敷衍的态度还是表明了 Linux 在今后一阵子会逐渐在国内扎根。因此在这种背景下，其实使用 Windows 和 Linux 其实没有多大区别。

![深度系统拥有漂亮的桌面和 Windows 般的易用，在某种程度上可以说与 Windows 无缝转接。它用 Wine 配适了国内大多数讨厌 Linux 的大厂的软件，甚至做的比这些大厂的原生 Linux 软件要好](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413155726.png)

Linux 作为主力机感觉和 Windows 差不多，那是不是可以无缝转接？其实不然，还是存在一些问题的，我后面会说。

## 选何种 Linux 系统

[Linux 官网](https://www.linux.org/ "Linux") 上列出的受欢迎的发行版有 24 款，但其实不止这么点。Linux 发行版多，其实主要分成三派：[红帽系(RHEL)](https://www.redhat.com "RedHat")，包括 [CentOS](https://www.centos.org/ "CentOS")、[Fedora](https://getfedora.org/ "Fedora") 等企业级系统；[大便系（Debian）](https://www.debian.org "Debian")，包括著名的 [Ubuntu](https://ubuntu.com "Ubuntu") 和国内的各种麒麟系统，主打个人电脑；[Arch 系](https://archlinux.org "Arch")，包括 [Manjaro](https://manjaro.org "Manjaro") 和各种新奇的妖魔鬼怪系统，主打的就是妖魔鬼怪。除了这三个派别三足鼎立，还有辽东公孙康之大蜥蜴 [OpenSUSE](https://www.opensuse.org "OpenSUSE")，交趾士燮之祖宗之法不可变 [Slackware](https://www.slackware.com "Slackware") 和南蛮孟获 [Gentoo](https://gentoo.org "Gentoo")，其余的不足道哉。

>
> Debian 叫做“大便”其实是个调侃罢了，毕竟人家和“大便”就差一个字母。
>

![Linux 官网上的 Linux 系统](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413160450.png)

至于这么多系统该选哪一个，那就是见仁见智的问题了。个人建议：

1. 大厂背书的。这就得排除各种小喽啰，因为人家保不齐就不维护了。例如 [CutefishOS](https://cutefish-ubuntu.github.io/ "CutefishOS") 刚出来就惊艳全场，被誉为国内最漂亮的系统，国内外享名，结果不久就停止维护了，甚至不发个通告，后被其他爱心人士接手维护。所以这类小作坊系统或者个人维护的小玩具就不要用来作主力了，否则大难临头欲哭无泪。
2. 生态面广的。生态面就是可用软件，现在 RHEL 系和 Debian 系都是生态比较好的，各个大厂提供 Linux 版软件都会适配这两个系统，也就是后缀为 rpm 和 deb 的安装包。这里 Arch 打包比较轻松，所以也许可以考虑 Arch。
3. 本地化好的。这要看个人，有些人可能习惯中文环境或者中国人软件的使用习惯，会选择比较本地化的东西。比如深度系统就是本地化做的很好的，还有 Ubuntu。Ubuntu 是在国内的名声很大，甚至对大多数人而言，Linux 等于 Ubuntu，反之亦然。

综上，其实也就是我列出的三大类别：RHEL 系适合企业使用，所以软件比较古老，个人使用比较体验可能不好，如果不添加其他源的话。Debian 系其实就靠 Ubuntu 及其衍生系统撑起来的名声，推荐使用 Ubuntu，个人桌面基本做到开箱即用，软件也还算可以。不过建议直接使用原版而不是使用国内的各种麒麟版本，麒麟是祥瑞，但在国产 Linux 系统里，基本是垃圾的代名词。如果国内出了某个新的麒麟系统，那猜“垃圾”绝对没错，人家就是想过过日子。Arch 系最突出的特点就是软件多、广，软件源新，基本上什么东西刚出来都可以在 Arch 上找到，这里说的是[AUR（Arch User Repository）](https://aur.archlinux.org/ "AUR") 。这样可选的 Linux 就是：

1. Fedora, 开瓢的 RHEL 系统，的小白鼠。有大厂背书，不会搞到一半人跑路。软件在某种程度上还是很新的，因为是小白鼠级别。说是小白鼠，其实不是说不好，而是很多新的特性都是在 Fedora 上先使用然后看情况转接到 RHEL 系统，看起来就是小白鼠。默认桌面是 GNOME, 桌面风格简约高效 —— 自称的，我觉得其实蛮合适，简约得难以置信。如果还是觉得不好，那 Linus Torvalds, Linux 之父用的就是 Fedora.
2. Ubuntu, 著名的 Linux 个人操作系统，虽然也有服务器版。这个基本不用多介绍，人站那就是大佬。默认桌面也是 GNOME. 与 Fedora 一样适合小白入手，而且可能比 Fedora 更适合。电脑出了啥问题一半在百毒上都可以搜到。
3. OpenSUSE, 德国人的 Linux 操作系统。德国人以严谨著称，该系统也和德国人一样呆板、严谨，很优秀但国内知名度不高。默认桌面是 GNOME.
3. Manjaro, Arch 系。包括 Arch 的优秀特性，但是去除了 Arch 入手难度高的缺点，是一个优秀的 Arch 发行版。如果觉得自己没能力驾驭 Arch, 但又想使用 AUR 上各种各样的软件，可以尝试 Manjaro —— 但是千万不要添加 AUR 国内源，一半的 Manjaro 都是因为这个挂的，使用正统 AUR 就行了，国内软件也有打包，没必要用国内源。默认有三个桌面：GNOME、KDE 和 XFCE，还有其他的社区维护桌面版本。
3. Arch Linux, 进阶版本的 Linux. 凭心而论，不适合小白使用，属于进阶用户的入门系统。没有默认桌面，或者说默认没有桌面。Arch 的用户手册称第二没人敢称第一 —— Gentoo 可能也敢称第二。基本上电脑遇到的问题都可以在用户手册上找到，相当于把 Ubuntu 遇到的问题集合到了一起。
3. Gentoo, 进阶版本的 Linux. Arch 进阶，Gentoo 则更疯狂，会使用 Arch 不一定会用 Gentoo, 但会 Gentoo 一般就不怕 Arch 烦人了。和 Arch 一样属于手册极其详尽的系统，但是没有点 Linux 基础和使用经验可能不习惯。
5. [Linux From Scratch](https://www.linuxfromscratch.org "LFS"), LFS. 高级版本的 Linux. 从头打造自己的 Linux 系统。走过几次安装手册，只要会做 Logo，就可以自己做一个国产的操作系统，然后去坑蒙拐骗了。

上面列出了 7 个发行版，前四者基本做到开箱即用，特别是 Ubuntu；后三者则是习惯 Linux 后的进阶版本，其中 LFS 可能最多做个玩具练练手，但其余两者可以作为主力使用，我使用的就是 Arch, 前期使用 Manjaro 作为过度。

## Linux 桌面

Linux 桌面不够稳定是事实，但主要还是看使用的人。我使用的是 Arch Linux，感觉还是很稳定的，特别是使用 Manjaro 期间，基本没有遇上什么问题。Linux 系统最怕的就是 Nvidia 卡（N卡）和游戏。如果在这方面没有什么特殊追求，一般而言还是很稳定的。上面介绍过，Linux 桌面主要是 GNOME 和 KDE, 还有各种分化的桌面 XFCE、Cinnamon、LXDE、Mate 等妖魔鬼怪和直接使用窗口管理器的平铺窗口 [i3wm](https://i3wm.org "i3")、[Swaywm](https://swaywm.org) 等。给桌面选择添加几个过滤条件，其实也就 GNOME 和 KDE 两种。GNOME 和 KDE 所占份额最大，其中 GNOME 有 RHEL 背书，KDE 背靠 Qt，没啥后顾之忧，其余的不是用爱发电就是高手的进阶桌面，不适合日常使用。当然，一些 GNOME 衍生的版本，例如 Mate 还是很值得玩味的。有人说 GNOME、KDE 太重。其实正常情况下，他们也没多少占用。

我的 Arch 使用的是 GNOME 桌面。Linux 桌面不论是 GNOME 还是 KDE 其实可能都做不到开始直接使用，因为每个人的审美都是不一样的，难免要进行一些调整。GNOME 的调整一般通过 [GNOME Shell 插件](https://extensions.gnome.org "GNOME Shell Extension") 调整，KDE 自带的样式调整就够了。调整桌面一般也会整桌面主题，就像手机的主题一样，这些微调会给人一种满足感，就像整自己手机一样。同样的，这种快乐一般诱发时间的流逝，所以很多刚接触 Linux 的人会莫名其妙花很多时间打扮自己的系统，然后有一种分享欲。一开始大家肯定不知道怎么调整才好，会自然而然地搜各种美化教程。不过说实话，习惯之后会觉得，其实这些没必要。但是这两个系统如果是从头搭建，一般都需要装插件或者调整。KDE 桌面调整的需要少一点，GNOME 不调整会觉得桌面很丑。

选好发行版和桌面基本上就可以安装使用了。一般而言，安装某个发行版最好的方式是看官方教程，看网上的教程很多都是相互借鉴的，甚至不知所云的，容易把系统搞坏。

# Linux 上的 Windows 替代

在国内使用 Linux 系统不可避免要碰到一些问题，不过国内的 Linux 生态逐渐得到完善，很多软件都有 Linux 原生版本了，比如百度网盘，可以直接到官网下载 Linux 版本。对 Linux 系统来说，很少有直接使用安装包安装的习惯，一般是依靠包管理器，例如 RHEL 系的 dnf、yum，Debian 系的 apt 和 Arch 系的 pacman. 对于 Arch 系的系统可以直接使用 [yay](https://github.com/Jguer/yay "YAY"), 一个 AUR 助手，可以完成 Linux 软件和很多用户打包软件的直接安装。

![Yay 搜索 baidunetdisk 的结果，结果显示的是可以下载的百度网盘软件，还会有简短的介绍](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413185910.png)

如果不知道哪些软件有没有 Linux 版本，可以上网搜搜或者直接到 AUR 上面查看有没有这个包。如果不喜欢使用命令行，现在的各种发行版都会有自己的软件商店。除了百度网盘还有 Linux 版本的有：

1. WPS for Linux. Linux 版本的 WPS，功能差别不大，可用，没有广告，可以登录账号。
2. QQ for Linux. 最新的 Linux 版 QQ，基于 Electron, 功能还算完善，使用过觉得不错。如果不喜欢可以选择其他基于 [Wine](https://www.winehq.org "Wine") 的版本。
3. Wechat for UOS. 统信系统定制版的微信。没啥可说的，也是基于 Electron. 功能应该还算完善吧，也有 Wine 版本可选。
4. Nutstore for linux. 坚果云 Linux 版。还行，不常用。
5. [Aliyunpan](https://github.com/liupan1890/aliyunpan "小白羊"). 阿里云盘小白羊版。
7. Neteasti Cloud Music. 网易云音乐 Linux 版本。
8. [QQ Music for linux](https://y.qq.com/download/download.html "QQ Music"). QQ 音乐 Linux 版本。
8. WemeetApp. 腾讯会议 Linux 版，不过只能在 X11 协议下分享屏幕，Wayland 暂时不行。
8. ...

不过对我来说，国产软件就像毒瘤，我不愿让它们污染我的电脑，所以是不会安装的。例如百度网盘、阿里云盘和迅雷、QQ、微信等。我的习惯是，开一个虚拟机，把这些毒瘤装上去，然后给它们 1G 内存，让它们自己折腾。我在 Linux 上使用虚拟机虚拟的是 Windows7，不是不想用 WindowsXP, 而是自己确实装不上。Windows7 的优点是即使内存占用很高还是很流畅。我之前虚拟过 Windows10，卡的不行。别说是在里边运行国产毒瘤了，就是在桌面上右键都要等半天，最后只能选择经历史考验的 Windows7 系统。使用 [QEMU](https://www.qemu.org "QEMU") 进行虚拟，也可以使用 [Oracle VM VirtualBox](https://www.virtualbox.org "vbox") 或者 [VMware](https://www.vmware.com "vmware")。QEMU 图的是轻量级，虚拟完可以写一个图标文件，把虚拟机放到桌面或者软件界面，开机启动挂载微信和 QQ 放到另一个工作区，时不时用一些国产毒瘤来下载东西。这里不用 Linux 版是因为，这些毒瘤鬼知道会拿点什么隐私东西。把他们放到虚拟机给点内存能活就行，它们可以在里边大展手脚，爱怎么折腾怎么折腾。

![GNOME 上用 QEMU 虚拟的 Windows7 系统和系统图标](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413191425.png)

![打开的 Windows7 系统和系统桌面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413191530.png)

对于使用 QEMU 虚拟 Windows 系统我后面打算也写一个教程。不过对 QEMU 的使用仅限于皮毛，因为不是科班出生，计算机硬件了解不多。除了这些，也可以在 Windows7 虚拟机上运行一些 Windows 独占的软件，给点内存就行。至于打游戏这种东西，Linux 确实带不动，所以最好给电脑来个双系统，以防万一想打游戏。

对于文字处理，例如 Microsoft WORD，可以用 WPS for Linux替代，我则使用 [VIM ](https://github.com/vim/vim "VIM") + [Pandoc](https://pandoc.org/ "VIM"), 先用 Markdown 写下文字，然后用 Pandoc 将 Markdown 转为 DOCX 格式，所以对 WPS for Linux 也就图个能看就行。对于 PDF 文件，一般使用 zathura 查看就行了。

![VIM 用 Markdown 写文章，通过 Pandoc 转换然后在浏览器中查看。左侧是 VIM 写的 Markdown 格式，右边是 Pandoc 转换的 HTML 格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413194332.png)

![zathura 查看 PDF 文件，设置了护眼模式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413194726.png)

文献管理用 [Zotero](https://www.zotero.org/ "Zotero"), 配合坚果云可以和手机的 [Zoo for Zotero](https://github.com/mickstar/Zoo-For-Zotero) 进行跨平台文献查看。

![Zotero 界面，可以配置暗色护眼](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413194952.png)

除了游戏以外，特别是对电脑性能要求高的游戏，比如我之前玩的 Arma3, 在 Linux 上安装 Steam 玩成 PPT 模式，Linux 可以应付生活中大部分工作。不过一些游戏，例如《饥荒》就不用担心完全可以带动。我觉得玩游戏还是用 Windows 系统比较好，Windows 和 Linux 不是不共戴天，而是可以互补的。我的电脑是双系统，一般只用 Linux, 打游戏就使用 Windows.

然后是 Linux 系统上的输入法，我用的是 [Fcitx5](https://fcitx-im.org "Fcitx5"), 手机上用 [Fcitx5 for Android](https://github.com/fcitx5-android/fcitx5-android "fcitx5 for Android") ，后者可以在 [F-Droid](https://f-droid.org "F-Droid") 和 Github 上下载。手机上的 Fcitx5 颇有谷歌输入法的风范，蛮好看。不使用国内的输入法是因为，国产软件实在恶心，感觉总要把人底裤扒得干干净净才好。使用 Fcitx5, 电脑的词库可以和手机的词库互通，还不用担心人民企业家们偷窥你的隐私。除了 Fcitx5，还有一个开源的输入法叫 [中州韵Rime](https://rime.im "Rime")。不过这个输入法（引擎）不是很适合普通人，并且有些行为确实反人类：长按输入字符或者键盘上写各种提示。这个输入法引擎很优秀，但是不是普通人能用的，而且看他原本也不是拼简体的 —— 可以，但不是本职，懒得折腾就直接使用 Fcitx5。

![Fcitx5 在电脑上的输入框，开了百度输入法的云输入，第二个候选字，有云输入还是很准确的](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413202808.png)

![Fcitx5 在手机上的界面，界面类似谷歌输入法，简洁明了](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202304/20230413202514.jpg)


# 后语

重新写博客，我打算写写消失几个月内做的一些事，包括：

1. QEMU 虚拟 Windows7 直接使用国产毒瘤，其中会包括使用 VirtIO 提供更好的体验。
2. Arch Linux 和 Windows 双系统无缝切换。
3. Arch Linux 安装 Nvidia 卡，并在需要的时候调用。
4. 五笔输入法快速入门。
5. 布努文字入门。

其中，布努文字草案虽出来很久了，但是我觉得都不太好，打算按照自己的方言整合一套文字。自己最近也在按自己整合的文字重新数字化一本布努文字典，不过一万多个词条太多了，可能会时不时在博客更新。此外还有自己写的一些小脚本，例如 VIM 上写 Markdown, 感觉如果有时间也会记记什么的。最近也没有在做什么，只是觉得做的东西不是很适合写下来罢了。
