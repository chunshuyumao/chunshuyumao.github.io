---
title: 安装 RoseTTAFold
hide: false
tags:
  - Linux
  - RoseTTAFold
categories:
  - RoseTTAFold
math: false
mermaid: false
date: 2023-04-30 08:13:40
excerpt: |
  RoseTTAFold 是华盛顿大学（University of Washington） Divid Baker 团队开发的一个蛋白质结构预测工具，2021 年在《科学》（*Science*）杂志发表。RoseTTAFold 基于三轨神经网络（3-Track neural network），预测可兼顾蛋白质序列模式、蛋白质内氨基酸相互作用和蛋白质可能的三维结构。本文介绍如何本地安装。
---

# 前言

RoseTTAFold 是华盛顿大学（University of Washington） Divid Baker 团队开发的一个蛋白质结构预测工具[@AccuratePredictionProtein2021]，2021 年在《科学》（*Science*）杂志发表。RoseTTAFold 基于三轨神经网络（3-Track neural network），预测可兼顾蛋白质序列模式、蛋白质内氨基酸相互作用和蛋白质可能的三维结构。

RoseTTAFold 开源，可到 GitHub 下载代码源码在本地搭建，也可在网上使用。Baker 团队搭建了一个在线版本，可以到 [RoseTTAFold server](https://robetta.bakerlab.org/ "RoseTTAFold server") 体验。

打开浏览器输入提供的网址，然后在左上角选择 Structure Prediction 下拉菜单。下拉菜单有三个选项：

- 提交（Submit）。提交要预测蛋白的序列，需要注册，我讨厌注册。
- Queue（队列）。查看提交预测的队列，提交线上预测的人很多得排队。
- 示例结果（Example Result）。展示预测蛋白的五个结果 3D 模型。

![RoseTTAFold server 使用](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430122158.png)

线上版本是免费的，全世界很多人都在排队使用，有条件可以选择本地搭建。

# RoseTTAFold 本地搭建

## 准备

RoseTTAFold 源码是开源的，可以直接到 GitHub 下载。不过在开始之前我还是要说说本地搭建的要求：

1. 一台 Linux 电脑。开虚拟机可能不行，因为这种运算，虚拟机太慢了，最好还是实体 Linux 机，或可以使用 Windows 系统的 [WSL](https://learn.microsoft.com/zh-cn/windows/wsl/install "WSL")（Windows Subsystem of Linux，Windows 的 Linux 子系统）。
1. 8G 内存。这是可以分配给 RoseTTAFold 的内存，不是电脑内存，建议电脑内存 8G 以上。我只试过源码自带示例在 8G 、64G 和 128G 内存跑 PyRosetta。当然，跑的都是 8 核，结果区别好像不大。8G 和 128G 都是 40 分钟左右，64G 没有单独做过，但是所有的测试都是 64G 做的，所以大差不差也是这个时间。反正一个小时应该是基本要求。生物嘛，一个小时很短的。
1. 2.6T 磁盘空间。RoseTTAFold 源码很小，但需要下载的模板和数据库压缩包有 402G，解压后有 2263.2G，即 2.3T。系统安装、解压后删除压缩包、安装 [conda](https://docs.conda.io/projects/conda/en/stable/index.html "Conda")，还保存一些数据，300G 只能算还过得去。所以整个电脑得有 2.6T 磁盘空间。
1. 电脑安装 conda/mamba。电脑安装的可以是 [minconda](https://docs.conda.io/en/latest/miniconda.html "Miniconda")、[anaconda](https://anaconda.org "Anaconda")。安装完之后可以安装 [mamba](https://mamba.readthedocs.io/en/latest/user_guide/mamba.html "Mamba") 作为 conda 的替代命令。

综上，如果要安装到自己电脑，需要花几个钱扩充一下硬盘空间。

我的建议是，有需要可以搭建，但自己电脑不行就不必勉强。可以询问自己的导师或者老师，看看学院有没有服务器。学校的服务器一般都很富裕。生信学生，一般学院都有服务器，毕竟生信做数据分析少不了。可能有的学校没有生信专业，所以还真没有服务器。

又或者，实验室有这个需求，可以问问老师，毕竟花个几百块钱买个硬盘不比动不动就几千上万的实验仪器便宜？RoseTTAFold 比较高的要求无非就是 Nvidia（英伟达）GPU（Graphical processing unit）和硬盘。前者可以不用，直接 CPU （Central processing unit）跑，后者现在还不算贵。至于内存，我觉得问题应该不大，实在不够可以花百十大洋买个内存条。

Linux 系统安装我就不演示了，这种教程我大清满大街都是，随便搜搜，或者可以翻我之前的博客。如果是学院、学校服务器就省了系统安装这一步。

## 下载源码

到 [RoseTTAFold GitHub](https://github.com/RosettaCommons/RoseTTAFold "RoseTTAFold GitHub") 主页查看如何下载。

如果使用 [Git](https://git-scm.com "Git") 下载源码，请确保自己的电脑安装了 Git。默认安装了 conda/mamba。

>
> 安装 conda/mamba 可以参考之前的博客 [服务器安装 RStudio Server](https://chunshuyumao.github.io/2022/03/18/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89%E8%A3%85RStudio-Server/) 
>
> 友情提示：conda 一定要换源，不然时间耗死你。最好安装 mamba 替代 conda
>

![RoseTTAFold GitHub 主页的安装教程](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430095355.png)

按照 GitHub 上的提示进行安装。当然，如果一帆风顺就不会有今天这篇文章了。

### 克隆（Clone）源码

使用 Git 克隆

```sh
$ pwd
/home/chunshuyumao/Downloads
$ git clone https://github.com/RosettaCommons/RoseTTAFold.git
...
$ cd RoseTTAFold
$ pwd
/home/chunshuyumao/Downloads/RoseTTAFold
```

### 创建环境

创建环境有两个选择：

- RoseTTAFold-linux.yml。Nvidia 卡支持的 Cuda 在 10 以上，一般是 11。
- RoseTTAFold-linux-cuda10.yml。Nvidia 卡支持的 Cuda 在 10 及以下。

如果太复杂，直接选择第一个。其实这个主要是看你的电脑有没有 Nvidia 卡，Linux 命令行输入 `lspci | grep -i vga`。

```bash
$ lspci | grep -i vga
02:00.0 VGA compatible controller: Matrox Electronics Systems Ltd. MGA G200e [Pilot] ServerEngines (SEP1) (rev 42)
```

上面的输出显示根本没有 Nvidia 卡。直接使用第一个创建环境算了。

```bash
$ pwd
/home/chunshuyumao/Downloads/RoseTTAFold
$ conda env create -f RoseTTAFold-linux.yml
```

创建完环境后，自己的 conda/mamba 下会有一个叫做 RoseTTAFold 的环境，可以使用 `conda env list` 查看。

安装完之后还有一个可选的 folding-linux.yml 环境，使用 [PyRosetta](https://pyrosetta.org/ "PyRosetta") 预测蛋白结构需要。RoseTTAFold 提供两种方式预测蛋白结构，一种是基于“点到点”（end-to-end,e2e）的方式，另一种是基于 PyRosetta。前者的预测效果可能不比后者。

使用 PyRosetta，我后面会介绍，就需要创建 folding 环境，只使用“点到点”的方式就不需要。

创建两个环境有点久，分别有 2G 左右。创建完毕查看当前环境：

```bash
$ conda env list
# conda environments:
#
base                     /home/chunshuyumao/.local/miniconda3
RoseTTAFold              /home/chunshuyumao/.local/miniconda3/envs/RoseTTAFold
folding                  /home/chunshuyumao/.local/miniconda3/envs/folding
```

### 下载第三方软件

```bash
$ pwd
/home/chunshuyumao/Downloads/RoseTTAFold
$ ./install_dependencies.sh
```

这个几个第三方包不大，下载很快。

### 下载数据包

创建 RoseTTAFold 环境的时候，可以再开一个控制台下载需要的数据包。按照 GitHub 上 README.md 下载和解压即可。

```bash
$ pwd
/home/chunshuyumao/Downloads/RoseTTAFold
$ # 解压有 1.2 G
$ wget -c https://files.ipd.uw.edu/pub/RoseTTAFold/weights.tar.gz
$ tar -zx -f weights.tar.gz

$ # uniref30 [46G]，解压有 182G
$ wget -c http://wwwuser.gwdg.de/~compbiol/uniclust/2020_06/UniRef30_2020_06_hhsuite.tar.gz
$ mkdir -pv UniRef30_2020_06
mkdir: created directory '/home/chunshuyumao/Downloads/RoseTTAFold/UniRef30_2020_06'
$ tar -zx -f UniRef30_2020_06_hhsuite.tar.gz -C ./UniRef30_2020_06

$ # BFD [272G]，解压有 1.8T
$ wget -c https://bfd.mmseqs.com/bfd_metaclust_clu_complete_id30_c90_final_seq.sorted_opt.tar.gz
$ mkdir -pv bfd
mkdir: created directory '/home/chunshuyumao/Downloads/RoseTTAFold/bfd'
$ tar -zx -f bfd_metaclust_clu_complete_id30_c90_final_seq.sorted_opt.tar.gz -C ./bfd

# structure templates (including *_a3m.ffdata, *_a3m.ffindex) [over 100G]，解压有 280G
$ wget -c https://files.ipd.uw.edu/pub/RoseTTAFold/pdb100_2021Mar03.tar.gz
$ tar -zx -f pdb100_2021Mar03.tar.gz
```

这几个文件下载的速度只能看命运了，可能晚上下载的速度快一点。上面的 `wget` 命令和 README.md 不大一样，多了一个 `-c` 命令。这是 `--continue` 的缩写，最好加上，相信我，不然你会等到死。

### 下载 PyRosetta（可选）

如果不想使用 PyRosetta，可以不进行这一步。当然，这也就意味着源码中的 `run_pyrosetta_ver.sh` 你无法使用。

#### 注册获取账户和密码

要使用 PyRosetta 需要注册一个账号。到[华盛顿大学](https://els2.comotion.uw.edu/product/pyrosetta "PyRosetta in UW")下载。根据我给的网址进入，可以看到如下界面。点击右下角的 ORDER NOW 即可。ORDER 后进入确认界面，再次点击 CHECKOUT NOW，进入登陆界面。

![到华盛顿大学软件下载界面下载 PyRosetta](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430103414.png)

![确认购买，购买不需要钱，因为可以免费使用。点击 CHECKOUT NOW](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430103528.png)

![登陆界面](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430103719.png)

如果没有账号，可以现场创建一个账号，选择 Register。后面的步骤就比较简单了，填写一下个人信息，至于地址机构什么的，随便填吧。后面根据邮件确认信息，然后会收到一封邮件。邮件里有自己可以使用的用户和密码，记住即可。

#### 安装 PyRosetta

打开 conda 配置文件 `~/.condarc`。这一步可以参考 PyRosetta [安装教程](https://www.pyrosetta.org/downloads "PyRosetta Installation")。

![官网安装教程](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430110137.png)

下面是我的 conda 配置，这是换源过后的。`remote_read_timeout_secs: 600.0` 设置 conda 下载超时时间 600 秒。因为是国外的软件下载速度肯定不好，所以超时时间可以设置久一点。

把自己注册后收到的密码和用户名填入下面的 PASSWD 和 USERNAME。

```bash
$ cat ~/.condarc
show_channel_urls: true
  default_channels:
    #- https://USERNAME:PASSWD@conda.rosettacommons.org
    - https://USERNAME:PASSWD@conda.graylab.jhu.edu
    - https://mirrors.aliyun.com/anaconda/pkgs/main
    - https://mirrors.aliyun.com/anaconda/pkgs/r
    - https://mirrors.aliyun.com/anaconda/pkgs/free
    - https://mirrors.aliyun.com/anaconda/pkgs/msys2

  custom_channels:
    conda-forge: https://mirrors.aliyun.com/anaconda/cloud
    msys2: https://mirrors.aliyun.com/anaconda/cloud
    bioconda: https://mirrors.aliyun.com/anaconda/cloud
    menpo: https://mirrors.aliyun.com/anaconda/cloud
    pytorch: https://mirrors.aliyun.com/anaconda/cloud
    simpleitk: https://mirrors.aliyun.com/anaconda/cloud

auto_activate_base: false
remote_read_timeout_secs: 600.0 
```

设置好之后在命令行中输入：

```bash
$ conda activate folding
$ conda install pyrosetta -y
```

下载过程难免有多次失败，因为网络不行。如果想做到无人值守安装，可以指将上面的安装命令改成：

```bash
$ conda activate folding
$ while ! conda insstall pyrosetta -y; do echo 'Try again...'; done
```

这样如果下载失败，电脑会自动帮你恢复安装，直到安装成功或者你放弃。注意，如果是自己的电脑，那千万不要让电脑休眠（Suspend）。如果是服务器，可以使用 [Tmux](https://github.com/tmux/tmux/wiki "tmux wiki") 连接服务器，然后后台运行。这是一个大话题，此处不进行解释。

如果安装成功，那就万事大吉了。

## 运行示例

安装完成就可以运行示例了，可以参考 RoseTTAFold 文件下的 README.md 文件。

首先进入 `example` 文件夹，然后运行 `run_pyrosetta_ver.sh` 或者 `run_e2e_ver.sh`。注意，没有安装 PyRosetta 软件就只能运行 `run_e2e_ver.sh` 脚本。

```bash
$ pwd
/home/chunshuyumao/Downloads/RoseTTAFold
$ cd example
$ mkdir -pv mytest
$ ../run_pyrosetta_ver.sh input.fa mytest
```

所有的输出文件都会放到 mytest 文件夹内，也可以把 input.fa 换成自己的蛋白质序列。

### 输出结果

#### end-to-end

结果会产生一个 PDB（Protein Data Bank）文件，文件放到输出目录的 model 文件夹内。例如我上面运行的输出目录是 `mytest`，所以 PDB 文件在 `mytest/model` 内。

#### PyRosetta

运行会产生 5 个模型文件，放到输出目录的 `model` 文件夹内，命名规则是 `model_[1-5].crderr.pdb`，头部包含一些误差信息，可以不用在意直接当作 PDB 文件使用。

### 输出结果可视化

输出结果是文件，怎么可视化查看？可以去 [RCSB](https://rcsb.org "RCSB") 查看。

浏览器打开 RCSB，选择“可视化”（Visualize）中的 Mol\*（MolStar）进行可视化。

![RCSB 网站选择“可视化”（Visualize）中的 Mol* 进行可视化](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430112202.png)

![点击右上角的“选择文件”（Select files...），添加自己的 PDB 文件，添加完毕选择“应用”（Apply）即可](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430112338.png)

![可视化结果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430113548.png)

至此，安装完成。马克思保佑！

## 一些问题

### \_\_mul\_\_ 属性不存在

> 
> While processing /home/chunshuyumao/Downloads/RoseTTAFold/example/mytest/pdb-3track/model0_1_0.05.features.npz: 'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute '\_\_mul\_\_'
>

按照官网的步骤搭建本地 PyRosetta，运行完后 model 文件夹内什么也没有。找到输出目录下的 log 文件夹，里边是程序输出的日志文件。

![查看日志，每一步都有输出](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430114206.png)

以 `stderr` 结尾的是错误日志，`stdout` 结尾的是输出日志。一般来说先看错误日志，然后再看输出日志。

查看 `run_pyrosetta_ver.sh` 文件可以看到脚本的运行顺序是：

1. 生成 MSA 文件，日志记录在 make_msa.std[err,out]；
1. 运行次级结构，日志记录在 make_ss.std[err,out]；
1. 查找模板文件，日志记录在 hhsearch.std[err,out]；
1. 预测距离和方向，日志记录在 network.std[err,out]；
1. 生成模型，日志记录在 folding.std[err,out]；
1. 择取最终模型。分成两步：
    1. 如果没有生成 NPZ 文件则先生成，日志记录在 DAN_msa.std[err,out]；
    1. 择取最终模型，日志记录在 pick.std[err,out]。

看看问题出在哪里：

![生成 NPZ 文件的步骤似乎有问题，DAN_msa.stderr ](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430144124.png)

![DAN_msa.stdout 好像也有问题](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430144319.png)

不要看 DAN_msa.stderr 就觉得它的问题严重性更强，其实不是，得通过报错信息来判断。其实 DAN_msa.stderr 只是发出了警告，不是错误。在程序员眼中，警告（Warning）并不是错误，大写也没用。

>
> 有个笑话：公路尽头是悬崖，警察在路口贴了一个警告牌，结果还是有人开车掉下去死了。警察一看，好家伙，原来是个程序员，他以为“警告”（WARNING）不是什么大问题，毕竟又不是“错误”（ERROR）。
>

这个警告不是程序出错的原因，再看 DAN_msa.stdout 文件。

```python
While processing /home/chunshuyumao/Downloads/RoseTTAFold/example/mytest/pdb-3track/model0_1_0.05.features.npz: 'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute '__mul__'
```

文件一直出现这个提示。其实这很让人费解，这是一个错误很明显，但是我不知道是哪里出现的问题。一个错误提示什么也没有，没有说哪个文件出了问题，没有说哪一行出了错误，也没说在干什么，只是说：俺在处理某某文件的时候发现某某对象没有某某特性。

呃，不得不说，这种情况就好像什么也没说。之前用 gThumb，GNOME 桌面一个看图软件，开了一个不存在的插件，gThumb 报错了，错误信息是：Oops! There is something wrong!

这！我都不知道该说点啥好：我当然知道有问题呀，你好歹提示我哪里有问题嘛，你这么一说，好像我不知道一样！

看来这两个报错信息有的一拼。

碰到问题不要去百毒搜索，我们先去官网看看是不是有人遇到了同样的情况。源码发布在 GitHub，先去 GitHub 查看。打开 RoseTTAFold 的 GitHub 主页，点击 Issues 查看问题。被解决的问题一般在 Closed，未解决的在 Open。

可以看到在 Open 的 Issues 里边，比较靠前的第三个问题就和我们遇到的一样。

![GitHub Issues 第三个问题和我们碰到的一样！](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430145621.png)

点进去查看具体信息。这个问题目前没有人回答，使用的版本和我的差不多，出错的情况差不多，也是 Python3.7 ，我当时就觉得可能是 Python 的问题。于是重新创建一个 Python3.6 环境运行，结果还是不行。

![查看具体信息。发现这个问题并没有人回答，问题是 23 年 3 月 18 号的，时间点和我们蛮近](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430145733.png)

自己动手，丰衣足食，直接看代码。我们运行的是 run_pyrosetta_ver.sh 文件，故此查看。找到输出 DAN_msa.std[err,out] 文件的那一行，调用的是 ErrorPredictorMSA.py，然后出错。那就简单，直接找到这个文件，在 RoseTTAFold/DAN_msa 文件夹下。

![查看 run_pyrosetta_ver.sh 文件，可以看到错误发生是在调用 ErrorPredictorMSA.py 文件之后，说明问题多半在这个文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430150624.png)

#### 查看 ErrorPredictorMSA.py 文件

查看文件的时候可以跟着 Python 的调用走，例如找到 `main()` 函数，然后跟着一步一步走。我看了一会之后发现真正调用的地方在第 159 行，调用了 pyErrorPred.process 函数。

至于是怎么看的，其实只要看看它的参数即可。在函数调用的上面几行出现了关键字 `features.npz`，这样这类文件自然只会在下面出现。不用担心，反正是找到了。

找到之后跟着这个函数进去，用 VIM 可以直接使用 `gd`（Goto Definition）进行查看。如果使用其他编辑器，可以找找编辑器的快捷键或者跳转按钮。

![ErrorPredictorMSA.py 文件真正解决问题的地方在这里，调用了这个函数](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430151405.png)

跳转之后我来到了 process 函数的定义：DAN_msa/pyErrorPred/featurize.py 文件的第 607 行。继续查看。接下来的查看是没有任何技巧的，因为报错给的信息基本等于没有。

我是暴力查找的：在 process 函数内选一个点，例如第 615 行，插入一行输出：

```python
print('-----------------------GOT YA----------------------------')
```

插入之后再运行 `run_pyrosetta_ver.sh` 脚本。错误信息出现在这个输出之后，说明报错位置在第 615 行之后；出现在这个输出之前就说明问题在这行之前。这样慢慢推进，输出结果在 DAN_msa.stdout 查看。

也可以使用夹逼法：选两个点，如果错误信息在两者之间就慢慢压缩。如果出现在前者前边，那就调整位置继续夹逼。同理后边一样。

![process 函数的定义位置，DAN_msa/pyErrorPred/featurize.py 第 607 行](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430152002.png)

好巧不巧，我还真找到了，而且花的时间很短，可能是运气好吧！

第 619 行的函数 extract_EnergyDistM 调用，跳转进去，来到本文件的第 53 行定义。看起来都没什么问题：这指的是代码的逻辑。直到看到第 80 行，这里出现了一个似曾相识的字眼 —— `__mul__`，问题多半就是这里了。

结合刚刚的信息来看，代码中的 iru 是没有这个 `__mul__` 属性的。我们需要确定一下它到底有没有这个属性。

```python
'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute '__mul__'
```

#### 确定 iru 的属性

#### 打印属性

Python 可以使用 dir 函数查看对象的属性和内部函数。

![使用 dir 查看 os 模块的属性和函数等](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430153417.png)

在代码中添加相应的输出部分：在 iru 下面插入几行，打印它的属性，插入代码是 `print(dir(iru))`。

```python
# Get an edge iterator
74         iru = graph.get_node(index1).const_edge_list_begin()
75         irue = graph.get_node(index1).const_edge_list_end()
76
77         print(dir(iru))
78         
79         # Parse the energy graph.
80         while iru!=irue:
81             # Dereference the pointer and get the other end.
82             edge = iru.__mul__()
83          
```

![添加 print(dir(iru)) 打印 iru 的属性](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430153636.png)

在交互模式可以直接使用 dir(iru) 打印，但是脚本中需要用 print 打印。修改之后再次运行脚本。

再次运行脚本的参数和刚开始的一样，因为这样更省时间，输出会保存到 DAN_msa.stdout 日志文件。

#### 查看日志内的属性

查看 DAN_msa.stdout 日志，里边重复打印了很多次属性，只需要看一个即可。可以确认的是，iru 的确没有 `__mul__` 属性。

```python
PyRosetta-4 2023 [Rosetta PyRosetta4.conda.linux.cxx11thread.serialization.CentOS.python37.Release 2023.16+release.942c01d5066fd96860b7d268702b832fe906a739 2023-04-12T15:05:51] retrieved from: http://www.pyrosetta.org
(C) Copyright Rosetta Commons Member Institutions. Created in JHU by Sergey Lyskov and PyRosetta Team.
/home/chunshuyumao/Downloads/RoseTTAFold/DAN-msa/models/smTr
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'arrow', 'assign', 'dereference', 'pre_decrement', 'pre_increment', 'valid']
While processing /home/chunshuyumao/Downloads/RoseTTAFold/example/mytest/pdb-3track/model0_1_0.05.features.npz: 'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute '__mul__'
```

![属性被打印到日志 DAN_msa.stdout 文件中，可以看到 iru 确实没有 \_\_mul\_\_ 这个属性](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430154333.png)

再看看代码注释是怎么说的：

```python
# Dereference the pointer and get the other end.
edge = iru.__mul__()
```

![代码对调用 __mul__ 的解释是 Dereference，也就是解引用](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430154706.png)

“解引用”，哈哈，不知道大家是怎么想的。写这个代码的人多半使用过 C/C++ 语言，因为 C/C++ 语言的解引用操作是 `*variable`。`variable` 是变量，例如 `iru`，解引用就是在变量前面加一个 `*`，这个符号同时也是乘法符号（multiple），在 Python 中的乘法操作重载用 `__mul__` 定义，解引用直接用这个符号，有点好玩。

它没有定义这个操作符我们无法使用。仔细看上面的属性发现，有一个属性是 `dereference`，恰好就是解引用的意思。搞不好就是？其实就是了。这应该是代码重构后没改过来的部分。于是我们把 `edge = iru.__mul__()` 改为 `edge = iru.dereference()`。

```python
######################################
# Fill the dist_matrix with energies #
######################################
aas = []
for i in range(length):
    index1: int = i + 1
    aas.append(pose.residue(index1).name().split(":")[0].split("_")[0])
    
    # Get an edge iterator
    iru = graph.get_node(index1).const_edge_list_begin()
    irue = graph.get_node(index1).const_edge_list_end()

    # Parse the energy graph.
    while iru!=irue:
        # Dereference the pointer and get the other end.
        #edge = iru.__mul__()
        # ↓↓↓↓↓↓↓↓↓修改这里↓↓↓↓↓↓↓↓↓↓↓↓
        edge = iru.dereference()
        
```

![修改后的代码](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430155525.png)

修改完再次运行脚本：

```bash
$ pwd
/home/chunshuyumao/Downloads/RoseTTAFold/example
$ ../run_pyrosetta_ver.sh input.fa mytest
```

修改完之后发现 mytest/model 下仍空空如也，再次查找问题。

### plus_plus 属性不存在

还是 DAN_msa.stdout 文件有问题，错误如下：

```python
While processing /home/chunshuyumao/Downloads/RoseTTAFold/example/mytest/pdb-3track/model0_0_0.15.features.npz: 'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute 'plus_plus'
```

![新的问题，似乎是没有 plus_plus 属性](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430161457.png)

不用看，继续在我们刚刚的代码找。

```python
101  # Move pointer
102  iru.plus_plus()
```

![刚刚修改代码的地方，第 102 行就调用了 plus_plus 函数](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430161623.png)

看注释写的是 Move pointer，函数又是 plus_plus：这是 C/C++ 语言移动迭代器/指针的方式，即 `++variable` 或者 `variable++`。`++` 不就是 Plus Plus 嘛！

查看刚刚的输出，iru 内有一个属性是 `pre_increment`，`pre_increment` 是前进没错了，就是不知道 `pre` 是啥意思。为了确认这是我们找的函数，我们需要查看这个函数的说明。

在 `iru.plus_plus` 上面一行添加 `print(iru.pre_increment.__doc__)`，查看函数的信息。这是 Python 查看函数说明的方式，函数说明一般集中在 `__doc__` 属性内。

![查看 pre_increment 函数的说明](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430162142.png)

重新运行脚本 `run_pyrosetta_ver.sh`，查看结果，结果在 mytest/log/DAN_msa.stdout 中。

```python
pre_increment(self: pyrosetta.rosetta.utility.graph.EdgeListConstIterator) -> pyrosetta.rosetta.utility.graph.EdgeListConstIterator

increment operator.  Point this iterator at the next element in the list.

C++: utility::graph::EdgeListConstIterator::operator++() --> const class utility::graph::EdgeListConstIterator &
```

![pre_increment 函数的定义](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430162459.png)

上面的输出很明显了，这就是我们要找的函数，用于移动指针/迭代器（Move pointer），而且是前加加（previous plus plus） `++variable`，可以说命名很规范了。

#### 修改 plus_plus 代码

找到目的函数，把 `iru.plus_plus()` 改成 `iru.pre_increment()` 即可。

```python
# Move pointer
#iru.plus_plus()
iru.pre_increment()
```

![把 iru.plus_plus() 改成 iur.pre_increment() 函数](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430162937.png)

随后重新运行脚本。至此问题结束！

![重新运行，model 文件夹出现了我们要的文件，问题解决！](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430163510.png)

# 后语

问题解决了，但是我没有在 GitHub 回复那位同样碰到问题的同志，谁有时间或者兴趣可以回复回复人家，毕竟人家有需要，而且等一个月了没人回应。我不回只是因为我不想登录自己的账号。

此外再调侃修改代码中遇到的事。

虽然写下 RoseTTAFold 很厉害，但是这个团队的人技术明显不大一样。

首先是编写 PyRosetta C++ 源代码的人，他写了函数的使用说明（\_\_doc\_\_属性），这是一种很好的写代码习惯，一个函数应该有函数说明，让别人知道自己是干啥的。

其次是写 Python 代码的人，在一些非正常操作，例如使用 `*` 作为解引用（调用 \_\_mul\_\_ 函数），应该写下注释给看代码的人知道这么做的目的。

上面两种操作都很专业，可以看出来技术很硬。

然后是编写 Shell 脚本的人，包括 run_pyrosetta_ver.sh 和 run_e2e_ver.sh 脚本，有一些操作做的很专业，但有些操作又很不应该。例如他知道先判断有没有生成相对应的文件，如果有就不需要重新运行程序，节约时间。但他同时又没有判断自己的程序有没有运行成功，直接让后面的脚本继续运行了。

```bash
106 ############################################################
107 # 6. Pick final models
108 ############################################################
109 count=$(find $WDIR/pdb-3track -maxdepth 1 -name '*.npz' | grep -v 'features' | wc -l)
110 if [ "$count" -lt "15" ]; then
111    # run DeepAccNet-msa
112    echo "Running DeepAccNet-msa"
113    python $PIPEDIR/DAN-msa/ErrorPredictorMSA.py --roll -p $CPU $WDIR/t000_.3track.npz $WDIR/pdb-3track $WDIR/pdb-3track 1> $WDIR/log/DAN_msa.stdout 2> $WDIR/log/DAN_msa.stderr
114 fi 
```

![第 110 行 "if $count" 那里做了检测，只有未生成自己想要的文件才运行下面的代码，避免浪费时间。可第 113 行有没有对运行的脚本结果进行判断，导致这里明明出现了问题，后面的代码仍然运行](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20230430/20230430170143.png)

这种错误可能让人做出错误判断，例如无法正确找到出错的地方。

这种错误是故意的还是无意的？应该不是无意的，而是故意的，因为在 RoseTTAFold 的 README.md 文件中，团队表示这两个脚本只是简单让我们知道这个程序怎么用而已。由此可知这两个脚本的目的只是为了演示，所以他们简化了步骤，也就让脚本健壮性差很多。

上面的脚本可以改成：

```bash
106 ############################################################
107 # 6. Pick final models
108 ############################################################
109 count=$(find $WDIR/pdb-3track -maxdepth 1 -name '*.npz' | grep -v 'features' | wc -l)
110 if [ "$count" -lt "15" ]; then
111   # run DeepAccNet-msa
112   echo "Running DeepAccNet-msa"
113   if ! python $PIPEDIR/DAN-msa/ErrorPredictorMSA.py --roll -p $CPU $WDIR/t000_.3track.npz $WDIR/pdb-3track $WDIR/pdb-3track 1> $WDIR/log/DAN_msa.stdout 2> $WDIR/log/DAN_msa.stderr; then
114     echo "run DeepAccNet-msa: Error!"
115     exit 1
116   fi
117 fi 
```

这样只要脚本没运行成功，后面的命令是无法执行的，而且根据输出可以明显看到是哪一步出了问题。同理，脚本里其他代码的运行都可以如此判断。

还有一点，就是我在文中调侃的报错信息：

> 
> While processing /home/chunshuyumao/Downloads/RoseTTAFold/example/mytest/pdb-3track/model0_1_0.05.features.npz: 'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute '\_\_mul\_\_'
>

这里用的是 `try/catch` 进行错误捕捉，用 `try/catch` 我觉得不合适，但是无所谓，因为这是个人习惯问题。我不喜欢 `try/catch`，他们没必要惯着我。可问题是，为什么错误信息不给个位置和文件？这个我非常不理解。

一般来说，如果选择报错，特别是这种直接不处理的异常，应该给个提示，例如：

```python
While processing `/home/chunshuyumao/Downloads/RoseTTAFold/example/mytest/pdb-3track/model0_1_0.05.features.npz` in file `/home/chunshuyumao/test.py`, 619: 'pyrosetta.rosetta.utility.graph.EdgeListConstItera' object has no attribute '__mul__'
```

这样出现问题，其他人就可以快速定位了。当然，我觉得罪魁祸首还是 `try/catch`。愿世间没有 `try/catch`。
