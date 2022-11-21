---
title: 服务器安装 RStudio-Server
hide: false
tags:
  - RStudio
  - Miniconda
categories:
  - 服务器
math: false
mermaid: false
date: 2022-03-18 15:06:56
index_img:
excerpt: 服务器 Rstudio Server 安装
---

# 前言

RStudio Server 是 RStudio 的服务器版。单机版其实也已经够用，可是有些时候我们需要进行迁移或者跨平台使用，重新安装一遍非常麻烦。我的服务器只有2G 内存，比不上我的电脑，没理由使用服务器不用自己的电脑。可惜服务器不用也是浪费，而且自己配置也是一种乐趣，所以就花了点时间。其实这也和我不愿意安装各种客户端有关。

折腾好一天才成功在服务器安装 RStudio Server。这里奉劝天下士，有事没事真别浪费时间在国内找网上的资料，要真的有问题不行了，去相关的技术论坛找问题或者好好跟着官网教程走，不然就是在浪费生命。

本次安装的平台和软件:

1. Miniconda3 Linux 64-bit。使用 conda 管理多版本 R 语言和 Python。
2. RStudio Server v2022.02.0+443。RStudio 的服务器版本，可以通过浏览器接入。
3. ECS 共享型 n4 单核2G内存服务器，Rocky Linux 8.5 版，兼容 CentOS7/8 和 RHEL8 及以上版本。

我会带大家一步一步踩坑，所有按我教程走的时候遇到问题不用紧张——毕竟俺就是这么来的。

# Miniconda3

R 语言是免费开源的编程语言，被广泛运用到科学计算领域，被称为科学家的编程语言。R 语言这年头最为人所知的能力就是画图和数据分析。大数据分析时代，R 与 Python 齐驱并进（虽然 Python 是后来者居上）。与 Python 相比，R 语言独有的特点是版本不兼容：R 语言一般不具有向前向后兼容性，低版本使用得了的<ruby>包<rt>package</rt></ruby>高版本一般使用不了。正因如此，你会看到很多学者的电脑安装各种版本的 R 语言。

在 Linux 上安装 R 语言可能直接安装到系统路径，想使用多个版本就容易起冲突。解决这个问题最普遍的思想是通过虚拟环境管理器管理 R 语言版本。

Python 我就不多说了。Python 最著名的包管理器是 PIP. 默认情况下只要安装了 Python 就会自带这个东西。PIP 提供了简单的包查询、下载、查找和卸载功能，一般不解决依赖问题。正常情况下不会有什么大毛病，一碰到科学计算包就不好看了。举个例子，如果我要安装 A 包，A 包里边使用了 B 包，PIP 安装就会出错。好心的话，PIP 会提醒你少了某种东西，可惜大多数时候它只会和你一样懵。

为了更好的解环境包依赖问题，有一群人开始思考某种新的包管理器，conda 由此而生。conda 成功解决了依赖问题——你没安装的东西它会帮你安装——并发展成 <ruby>包管理系统<rt>package management system</rt></ruby>、 <ruby>环境管理系统<rt>environment management system</rt></ruby>，而且还兼管多种语言的版本管理——当然最常用的就是 R 和 Python.

conda 一般不能单独安装，想要使用它需要安装 Anaconda。Anaconda 和 Python 同祖同宗，都是大蛇。Anaconda 预安装上百个科学包，基本不用愁。可惜大多数我们都用不上。用不上又得安装，动辄几个 G 的内存看起来很恶心。鉴于此，Miniconda 横空出世：它也使用 conda 解决包依赖问题，但不像 Anaconda 什么都安装，它只保留最少的软件包，剩下你需要啥就装啥。对于内存捉襟见肘的小伙伴来说，这简直是福音。因此我打算安装 Miniconda3.

## 安装 Miniconda3

到官网[^1]下载对应系统的版本。我要安装到 Linux 服务器，所以选择 [Miniconda3 Linux 64-bit](https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh) 。使用你喜欢的下载方式下载，我用 axel.

```powershell
axel -n 32 https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

![选择版本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181615732.png)

下载完之后，使用 `ls`查看自己的目录下有没有以下脚本。如果没有需要继续下载。

![下载的脚本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181628572.png)

给予脚本可执行权限：

```shell
chmod u+x Miniconda3-latest-Linux-x86_64.sh
```

然后执行

```shell
./Miniconda3-latest-Linux-x86_64.sh
```

![执行](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181631221.png)

![一直 Enter 回车](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181631078.png)

![选择 yes 确认安装](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181632355.png)

![选择安装位置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181637069.png)

安装位置通常是你的家目录，例如我的家目录是 `/home/chunshuyumao`，所以安装位置是 `/home/chunshuyumao/miniconda3`，你也可以在 `>>>`之后修改自己的路径。建议还是安装到自己的家目录下。

安装完毕它会问你是否激活 conda 环境，选择 yes 会在每次打开控制台时自动激活 conda 的默认环境 base。如果不想默认开启就选择 no. 如果无意中选择了 yes, 也可以通过输入 `conda config --set auto_activate_base false` 取消激活。一般我选择 yes，然后再输入命令取消。

![默认激活](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181736554.png)

![选择 yes](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181740692.png)

安装完之后重新登陆控制台可以看到 conda 命令可以使用了，或者不用重启登陆，只需输入

```shell
source ~/.bashrc
```

其中 `~/.bashrc` 是你的 shell 的配置文件，我使用的是 zsh, 所以输入的是 `source ~/.zshrc`。输入 `conda -V` 查看安装是否成功。

![查看版本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181743461.png)

## 配置国内源

conda 不配置国内源速度会很慢，输入以下代码配置国内源：

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
```

配置国内源之后安装 mamba —— mamba 可以看作是更快的 conda. 由于 conda 没想到自己发展那么快，随着包越来越多，解析依赖的速度越来越慢，对于一些大型的包需要解决的依赖更多——这时候 conda 显得力不从心，于是诞生了 mamba 这个更快的包管理。不想了解这段历史的话直接安装 mamba 就对了。安装之后进行简单的升级，确认一下

```shell
conda install mamba -y
mamba update conda -y
conda update mamba -y
```

## 安装 R 语言

conda 通过创建多个<ruby>环境<rt>encironment</rt></ruby>管理语言的版本，要安装多版本需要创建环境。后面使用的 Seurat 包需要 R 语言的版本高于 4.0 ，所以创建一个高版本的 R 语言环境：

```shell
conda create -n r4.1.2 -y
```

> `-n` 指定环境的名字，为了方便区分，我直接命名为 `r4.1.2` ，意思是要安装 4.1.2 版本。`-y` 是 yes, 就是直接确认创建，不用再询问。

![创建 R 语言环境](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181804758.png)

激活环境，输入 `conda activate r4.1.2`。如下图，右边提示现在是 `r4.1.2` 环境

![激活环境](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181805983.png)

想使用哪个环境就激活那个环境。

> 如果想关闭环境，使用 `conda deactivate`

安装 R 语言 4.1.2 版

```shell
mamba install r-base==4.1.2 -y
```

安装完毕输入 `R` 回车，查看是否有问题。如果进入以下界面就说明安转成功，输入 `quit()` 退出 R 语言的交互界面。

同理，想要安装其他版本的 R 语言，需要创建一个新环境，然后激活它再安装版本——使用 `==` 指定语言版本，不指定的话默认安装最新版。

![检验安装是否成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181842541.png)

退出的时候它会问是否保存工作环境，一般不用保存，输入 `n` 即可

![不保存工作环境](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181843681.png)

## 安装 Python 

创建新环境，安装最新版 Python：

```shell
conda create -n py3.10 -y
mamba install -n py3.10 python==3.10 -y
```

这里在 `install` 的时候没有激活 py3.10 环境，所以需要使用 `-n` 指定安装 Python3.10 的环境。

安装完之后输入

```shell
conda info --envs
```

查看我们现有的环境

![现有的环境](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181849999.png)

可以看到，我们现在有三个环境: base, py3.10, r4.1.2。其中激活的是 py3.10。

> 哪个环境是激活的会有一个 `*` 标注。

# 安装 RStudio Server

## 安装

接下来到官网[^2]安装 RStudio Server. 选择你的系统，我用的是 RHEL 系列，点击第一个系统进入安装教程。

![选择系统](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181852320.png)

![安装教程](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181853918.png)

首先是下载安装包。使用 `wget` 就按照官网说的下载，使用 `curl` 就用 `curl` 下载，使用 `axel` 就用 `axel` 下载，下面三种下载方式选一种：

```shell
wget https://download2.rstudio.org/server/centos7/x86_64/rstudio-server-rhel-2022.02.0-443-x86_64.rpm
curl -O https://download2.rstudio.org/server/centos7/x86_64/rstudio-server-rhel-2022.02.0-443-x86_64.rpm
axel -n 32 https://download2.rstudio.org/server/centos7/x86_64/rstudio-server-rhel-2022.02.0-443-x86_64.rpm
```

确认下载成功：

![确认下载成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181856163.png)

安装：

```shell
sudo yum install rstudio-server-rhel-2022.02.0-443-x86_64.rpm -y
```

![安装完毕](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181859478.png)

允许自启动，并启动它：

```shell
sudo systemctl enable rstudio-server
sudo systemctl start rstudio-server
sudo systemctl status rstudio-server
```

![一般会启动失败](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181900768.png)

不用担心，启动失败很正常。输入

```shell
$ sudo rstudio-server verify-installation

TTY detected. Printing informational message about logging configuration. Logging configuration loaded from '/etc/rstudio/logging.conf'. Logging to '/var/log/rstudio/rstudio-server/rserver.log'.
Path to R not specified, and no module binary specified; Invalid R module ()
```

上面显示的错误是找不到 R 语言的路径，这个简单，我们配置一下 R 语言的路径。 

```shell
sudo vim /etc/rstudio/rserver.conf
```

按 <kbd>Shift</kbd>+<kbd>g</kbd> 转到最后一行，按 <kbd>o</kbd> 输入

```shell
rsession-which-r=/home/chunshuyumao/miniconda3/envs/r4.1.2/bin/R
```

其中，`/home/chunshuyumao`写你的家目录，这里是我的家目录， miniconda3 安装在哪里就使用哪个目录。我创建的 R 语言环境在 miniconda3  <ruby>虚拟环境目录<rt>envs</rt></ruby>下的 `r4.1.2`。如果你不确定你的 R 语言在哪里，可以激活环境之后使用 `which R` 查看，例如：

![which R](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181912027.png)

上面图中的 `~` 代表 `/home/chunshuyumao` 也就是你的家目录。

配置完之后，按一下 <kbd>Esc</kbd> 键，然后输入 `:wq` ，回车就可以了。再次输入 `sudo rstudio-server verify-installation` ，发现仍然有问题。

![修改路径](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181914017.png)

配置库路径

```shell
sudo vim /etc/rstudio/rserver.conf
```

按 <kbd>Shift</kbd>+<kbd>g</kbd> 转到最后一行，按 <kbd>o</kbd> 输入

```shell
rsession-ld-library-path=/home/chunshuyumao/miniconda3/envs/r4.1.2/lib:/home/chunshuyumao/miniconda3/envs/r4.1.2/lib/R/lib
```

把 `/home/chusnhuyumao` 改成你的家目录，`r4.1.2`改为你的 R 语言虚拟环境名即可。

按一下 <kbd>Esc</kbd> 键，然后输入 `:wq` ，回车退出。

重启试试:

```shell
sudo systemctl restart rstudio-server
sudo rstudio-server verify-installation
```

出现下面提示说明你已经成功

```shell
Server is running and must be stopped before running verify-installation
```

如果提示 `error while loading shared libraries: libssl.so.10`，请跳到 <a href="#prob">4</a>。

RStudio Server 默认监听 8787 端口，可以使用 `sudo netstat -an4p | grep 8787` 查看 rserver 是否启动

![查看端口](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181930508.png)

> 如果提示没有 `netstat` 这个命令，请先安装 `sudo yum install -y net-tools`

开启防火墙的话，开放一下 8787 端口：

```shell
sudo firewall-cmd --add-port=8787/tcp --permanent
sudo firewall-cmd --reload
```

## 测试登录

浏览器输入 `IP:8787`，这个 IP 是公网 IP 。登陆使用的是你服务器的帐号和密码。

> 如果你在虚拟机上尝试，请使用虚拟机的 IP，使用 `ip addr` 查看， inet 之后就是你的 IP

![登陆](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181940261.png)

![登陆成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181941757.png)

Rstudio Server 搭建完毕。

# <a name="prob">一些问题</a>

来到这里的都是因为这条错误：

```shell
/usr/lib/rstudio-server/bin/rsession: error while loading shared libraries: libR.so: cannot open shared object file: No such file or directory
```

这和 OpenSSL 有关。我在 Rocky Linux 8.5 遇到的问题，使用 CentOS7 虚拟机没有碰到。查看库目录存在 libssl 的各个版本，创建软链接想要骗过系统发现不行，最后使用 yum search 看到了`compat-openssl10` 包，看标签应该是兼容包，安装之后发现没问题，就此记录。

安装 `compat-openssl10.x86_64`后再次重启 RStudio Server 服务。

```shell
sudo yum install compat-openssl10.x86_64 -y
sudo systemctl restart rstudio-server
sudo systemctl status rstudio-server
```

# 安装 Scanpy 和 Seurat

Scanpy 和 Seurat 是数据分析使用的，个人安装，可以不用看。

## 安装 Scanpy

到 Scanpy 官网[^3]查看下载 Scanpy 教程

![安装 Scanpy](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203181958423.png)

激活 Python 环境安装：

```shell
conda activate py3.10
mamba install seaborn scikit-learn statsmodels numba pytables -y 
mamba install python-igraph leidenalg -y
pip install scanpy -i https://pypi.tuna.tsinghua.edu.cn/simple
```

> 请注意，一定要使用 pip ，不是 pip3 。

安装完毕，输入 `python`进入交互界面， `import scanpy` 成功就说明安装好了。

## 安装 Seurat

安装 Seurat[^4] 比较简单，激活 R 语言环境:

```shell
conda activate r4.1.2
R
install.packages('Seurat')
install.packages('dplyr')
install.packages('patchwork')
```

提示选择镜像源，可以选择北京——我这里是 17。选完直接回车。

![镜像源](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182010716.png)

Linux 安装 Seurat 的时候会进行源码编译，用时较长，需要有耐性。如果是 Windows 安装就比较快，直接二进制安装。

编译 Seurat 到一半提示

```shell
x86_64-conda-linux-gnu-c++: fatal error: Killed signal terminated program cc1plus
```

这是因为我的服务器只有 2G 内存，编译需要大量内存，显然内存不足。查看一下发现，我的服务器内存小就算了，还没有<ruby>交换分区<rt>swap</rt></ruby>，不卡死才怪。生成一个空文件，临时作为<ruby>交换分区文件<rt>swap file</rt></ruby>：

```shell
sudo dd if=/dev/zero of=/swaptmp bs=4M count=1024
```

解释一下，`swaptmp` 是<ruby>交换分区<rt>swap</rt></ruby>的名字，随便取。位置在 `/` 根目录下。
`bs`是 `blocksize`，就是区块大小，不用管。
`count` 是写入的区块个数，我写入 1024 块。

通过上面的命令，我创造了 4x1024M( 也就是 4G ) 大的文件。 为什么是 4G ？因为我的内存为 2G, <ruby>交换分区<rt>swap</rt></ruby>最好不超过内存的 2 倍。

> 内存指的是物理内存，不是硬盘空间，我的配置是 2G 内存 + 40G 硬盘空间

完成之后格式化并启用<ruby>交换分区文件<rt>swap file</rt></ruby>：

```shell
sudo mkswap /swaptmp
sudo swapon /swaptmp
```

继续编译 Seurat .

编译完不想保留<ruby>交换分区文件<rt>swap file</rt></ruby>可以卸载：

```shell
sudo swapoff /swaptmp
sudo rm -rf /swaptmp
```

不过依我看，如果你的内存不够编译 Seurat，索性留着<ruby>交换分区<rt>swap</rt></ruby>也许更好。想要保留<ruby>交换分区文件<rt>swap file</rt></ruby>，请进行以下配置：

```shell
sudo vim /etc/fstab
```

按 <kbd>Shift</kbd>+<kbd>g</kbd> 转到最后一行，按 <kbd>o</kbd> 输入

```
/swaptmp swap swap defaults 0 0
```

按一下 <kbd>Esc</kbd> 键，然后输入 `:wq` ，回车退出。

# 测试

登陆你的 RStudio Server. 输入

```
library(Seurat)
library(dplyr)
library(patchwork)
```

没有报错就说明成功了。

![测试](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182028913.png)

<kbd>Ctrl</kbd>+<kbd>l</kbd>清空交互窗口，新建一个 R 文件，输入以下内容：

```R
data=c(1, 2, 4, 8)
names=c("C++", "C","Python", "R")
cols=c('black', 'blue', 'red', 'green')

pie(data, labels=names, col=cols)
```

全选，点击界面右上角的 <ruby>运行<rt>run</rt></ruby>。

![新建文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182033434.png)

![运行](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182035833.png)

![效果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182035471.png)

点击 <ruby>工具<rt>Tools</rt></ruby> 选择 <ruby>全局选项<rt>Global Options</rt></ruby> ，选择 <ruby>包<rt>Packages</rt></ruby>，修改 <ruby>首要 CRAN 仓库<rt>Primary CRAN repository</rt></ruby> 为离你最近的地方。选择 Python 修改版本为你喜欢的版本——它会自动检测你服务器的 Python 版本。

![设置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182036919.png)

![修改镜像](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182037629.png)

![Python 版本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203182039945.png)

到这里，我们安装完 RStudio Server 还进行了简单的配置，完美收工。

# 后语

原本我不想使用 RStudio Server，最开始的想法是安装 R 语言，然后 VS Code 远程连接。之前在自己的虚拟机玩过，可以完美运行。因为没有图形界面，运行之后会产生一个 PDF 文件，只需要在 VS Code 安装一个 PDF 预览插件就可以了。不料今天各种问题：PDF 预览不了，R 语言的提示用不了。最后我都要疯了。配置 RStudio Server 的时候上不去官网查看教程，于是上网搜。真是倒霉的一天，网上一群人不知所云。最后还是逛国外网站才解决问题。真是有事没事别用国内的搜索引擎——你永远找不到自己想要的答案，我都不想吐槽了。

VS code 的设置真的反人类——至少反我。我都不知道哪跟哪，最后直接放弃了 VS Code. 我还留着 VS Code 仅仅是因为它的智能提示。不过我已经不玩 C/C++ 了，看来没必要留着这个东西。相比之下，我更愿意 VS Code 使用像 Sublime Text 那样的 JSON 配置格式——虽然我知道它也有，只是它的 JSON 格式让我感觉在写 C# ，真的不舒服。鬼知道，别人用都没问题就我有问题？

目前对我来说还有疑惑的是 Server 版不能安装包，只能使用控制台安装。我不是很了解，因为之前没有用过 RStudio. RStudio Server 的 Python 智能提示比较慢，可能和服务器有关吧，需要再鼓捣鼓捣。除此之外应该没啥大问题了。不清楚这个服务器顶不顶得住大规模运算——虽然未必有。我也听过 Jupter NoteBook 可惜自己玩不会。

内存不够用着实让我眼前一黑。当时正编译，远程控制台直接卡崩，其他控制台瞬间动不了。我以为是自动挂起，结果怎么都连不上服务器，赶忙上阿里云控制台，发现有一个严重警告，没看懂。当时以为是安装问题，登陆服务器再试安装 Seurat 两遍都是强行断连接。最后瞟了一眼才发现是内存不足被系统强行杀掉。一直以为内存不是大事的我才意识到，我的服务器只有 2G 内存。当初学的 `dd` 那点知识终于用上了，看来 swap 分区还是有必要的，特别是老旧、内存不大的机子。

安装 Seurat 的一些经历也奇怪。之前有一阵子安装 miniconda3 ，配置 Seurat 的时候一直安装不了 Stringi 包，差点没怀疑人生——因为之前安装都没问题。结果第二天又可以安装了。我就不信这个邪，当晚再试——果然又安装不了。最后估摸着发现，晚上安装不了，白天可以？有点意思，可能这个包昼出夜伏。开个玩笑，可能那时候服务器顶不住太多的请求，又或者 DNS 污染，反正就这么奇葩。

# 相关网址

[^1]: https://docs.conda.io/en/latest/miniconda.html
[^2]: https://www.rstudio.com/products/rstudio/download-server/
[^3]: https://scanpy.readthedocs.io/en/stable/installation.html
[^4]: https://satijalab.org/seurat/
