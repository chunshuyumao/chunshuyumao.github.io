---
title: Chevereto 配合 PicGo 搭建自己的图床
hide: false
tags:
  - 图床
  - Chevereto
  - PicGo
categories:
  - 博客
math: false
mermaid: false
date: 2022-03-31 16:24:12
index_img:
excerpt: 哈哈，我居然忘了我还有一个服务器？
---

# 前言

前一阵子，自己的图床没了觉得很可惜，当时还想，要不要去租个网络存储服务？等到冷静下来之后才想起：我有一个服务器来着！真是神奇。

有了想法自然要付诸行动，准备搭建自己的图床。之前我曾试着手动搭过 <ruby>LNMP<rt>Linux Nginx MariaDB PHP</rt></ruby>，可惜失败了。此外我还在自己的服务器里装过 OpenResty，所以还得进行一些清理工作。不得不说，当初搭建环境都是一次性代过，没有给服务器来个快照或者备份什么的，导致可能乱装了点啥——不过大多数时候我都会选择卸载掉自己安装的东东，除非哪个配置文件我忘了。

> LNMP 原来是指 Linux、Nginx、MySQL 和 PHP。只不过我用的是 MariaDB，所以没带上 MySQL.

# 准备

搭建图床服务的环境如下：

1. Linux 服务器，我用的是 Rocky Linux 8.5, 同等于 RHEL 8 和 CentOS 8。因为安装完成之后我才写的博客，所以这次用 Docker 下的 Rocky Linux 8.5 演示，勉强看作一个虚拟机吧。
2. [LNMP](https://www.lnmp.org/ "LNMP")，一键安装环境脚本，比手动简单多。我手动搭建过，各种配置太麻烦。这里提供一键安装它不香吗？
3. Chevereto，一个图床自建程序。
4. PicGo, 选择性。

下面我们慢慢介绍。

## LNMP 安装

LNMP 是一个网站服务器框架。简而言之，网络上很多东西都建立在这个框架基础之上。只要搭好这个框架，除了图床还可以快速搭建博客、网站等等。背景什么的咱就不介绍了——毕竟我也不是行家。

登陆自己的服务器，下载一个叫做 screen 的程序。简单说一下，使用 SSH 登陆服务器久不操作连接会自己断开。当然，为什么我们不操作呢，是吧？其实还是有可能的，比如安装软件，特别是 Linux 系统，有些时候编译某些东东的时间很长，长时间不操作 SSH 链接会自己断开。在 Linux 系统中，我们的操作基本都来自操作终端——就是我们的命令行——如果连接断开，终端就终止了。终端终止，我们的所有操作就都终止了。这样我们不得不重试，重试之后又断开，反反复复。

为了避免这种情况，我们使用 screen 命令创建一个独立于当前终端的空间，即使我们的连接突然断了，命令仍然被执行。这就是保命的东西。我们的操作基本都需要超级管理员权限，所以索性切换到超级管理员——root

```shell
su - root
yum install screen -y
```

安装完成之后使用 `screen -S lnmp` 创建一个叫做 lnmp 的空间。下载 [LNMP](https://www.lnmp.org/ "LNMP") 一键安装脚本，然后解压，进入自己解压的文件夹：

```shell
axel -n 32 http://soft.vpser.net/lnmp/lnmp1.8.tar.gz
tar zxf lnmp1.8.tar.gz
cd lnmp1.8
```

你可以选择自己喜欢的下载方式，我喜欢用 Axel 。

![下载解压](http://101.200.84.36/images/2022/03/31/202203311924121.png)

使用 `ls` 查看你的目录，输入 `./install.sh` 进行安装。如果你用的和我一样，都是 Rocky Linux, 那么恭喜你，你中奖了，喜提错误警告：

```shell
Unable to get Linux distribution name, or do NOT support the current distribution.
```

这是因为这个脚本没有为 Rocky Linux 准备。现在有两个选择：选择其他发行版，自己动手改脚本。我选择后者。

输入 `vim include/main.sh` 打开脚本，按 <kbd>/</kbd>，输入 Get_Dist_Name 找到目标函数，如下。回车。

在 `if` 后面添加

```shell
elif grep -Eqi "Rocky Linux" /etc/issue || grep -Eq "Rocky Linux" /etc/*-release ;then
    DISTRO='CentOS'
    PM='yum'
```

![打开脚本](http://101.200.84.36/images/2022/03/31/202203311928363.png)

![修改配置](http://101.200.84.36/images/2022/03/31/202203311935014.png)

这其实是让脚本以为自己在 CentOS 系统上运行——我说过 Rocky Linux 兼容 CentOS ，所以没必要担心。按 <kbd>Esc</kbd> ，输入 `:wq` ，回车，保存退出。重新执行脚本。

重新执行之后脚本会让你选择自己的数据库，如果知道的话就选择，不懂的直接默认。我选择最新的 MariaDB 10.4.19，所以输入 10. 后面设置数据库密码。

![选择数据库类型](http://101.200.84.36/images/2022/03/31/202203311940652.png)

![设置密码](http://101.200.84.36/images/2022/03/31/202203311943548.png)

安装 PHP。这就不是版本越高越好了，暂且选择 7.1 吧，因为我现在的数据库和 Chevereto 支持的就是这个版本。后面的那个内存分配器随便选择，也可以默认不选，我选择 2 。然后回车安装。

![PHP版本](http://101.200.84.36/images/2022/03/31/202203311946674.png)

到这里，基本可以洗洗睡了——会安装很久。我的服务器在安装过程中 CPU 使用率直接飙到 90% 以上，基本使劲浑身解数来解决编译问题，最后还是花了 126 分钟，简直慢到死。不过现在是在自己的电脑演示，速度还是挺快的。

如果在等待的过程中，你的 SSH 连接断开了，你可以再次登陆自己的服务器，然后用 `screen -r lnmp` 恢复原来的工作。如果你忘了自己给取的名字是啥，使用 `screen -list` 查看。等到运行结束之后，可以直接用 `exit` 退出 screen .

我们使用脚本的另一个好处是，如果自己的环境配错了，可以修改。比如我安装了 PHP 高版本，但是需要的是低版本，可以是使用 `./upgrade.sh` 进行修改。手动的话，比较麻烦。同理，如果想要卸载，也可以使用我们下载的 LNMP 中的脚本完成。也可以去 [LNMP](https://www.lnmp.org/ "LNMP") 脚本官网查看教程。

![安装完成](http://101.200.84.36/images/2022/03/31/202203312009531.png)

安装完成，使用了 22 分钟，比服务器强多了。我没有域名，所以就不搞虚拟主机了，直接上 IP.

看最后几行，上面提示 nginx、php-fpm 等都没有运行——按理来说是应该运行的，不过我用的 Docker, 所以不行而已。如果使用虚拟机或者服务器，通常应该没问题。例行公事，给他们开机自启的权限：

```bash
systemctl enable nginx
systemctl enable php-fpm
systemctl enable mariadb

systemctl restart nginx
systemctl restart php-fpm
systemctl restart mariadb
```

这时候，`/home` 目录下会多出几个 www 开头的用户，其中 wwwroot 目录下的 default 就是我们的图床位置——可以随意改变位置，但是我只按照这个目录演示。

可以看到 default 目录下有一个 phpmyadmin 文件夹，待会我们会访问它。

![phpmyadmin](http://101.200.84.36/images/2022/03/31/202203312027429.png)

打开配置文件，找到 server  有 `listen 80` 的那一部分配置。在 `include enable-php.conf` 后面添加以下配置：

```bash
vim /usr/local/nginx/conf/nginx.conf
```

```bash
location / {
    try_files $uri $uri/ /index.php?$query_string;
}	

location ~ [^/]\.php(/|$) {
    fastcgi_pass unix:/tmp/php-cgi.sock;
    fastcgi_index index.php;
    include fastcgi.conf;
}
```

![配置](http://101.200.84.36/images/2022/03/31/202203312033765.png)

注意配置里的一些字段，例如后面有的图片格式后面的括号显示有效期 ( expires ) 是 30 天。你可以修改自己的。

可以把 servername 改为自己的服务器公网 IP。

修改完之后，在浏览器中输入 `http://ip/phpmyadmin` 登陆创建数据库，用户是 root, 密码是你刚刚创建的，我的是 chunshuyumao.

> http://ip/phpmyadmin 中的 IP 是指你的服务器公网 IP，而不是这两个字符。

![登陆选择数据库](http://101.200.84.36/images/2022/03/31/202203312040197.png)

![创建图床数据库](http://101.200.84.36/images/2022/03/31/202203312043700.png)

创建数据库的时候，编码选择 utf8_general_ci . 我对数据库不是很了解，不能给专业的解释，只能告诉你这样避免乱码。

## Chevereto

### 安装

准备安装 [Chevereto](https://github.com/rodber/chevereto-free "Chevereto" ). 到 GitHub 下载压缩包或直接使用 `git clone` ，[官网](https://chevereto-free.github.io/ "Chevereto官网")了解教程。

```shell
cd /tmp
axel -n 32 https://github.com/rodber/chevereto-free/archive/refs/heads/1.6.zip
```

下载完之后解压、移动到 default 目录：

```shell
unzip chevereto-free-1.6.zip && mv chevereto-free-1.6/* /home/wwwroot/default/
```

我下载的是 1.6 版，解压的时候需要看自己下载的是第几版，不要照抄我的步骤。解压之后 default 目录应该长这样：

![default 目录](http://101.200.84.36/images/2022/03/31/202203312106787.png)

现在可以到浏览器直接输入你的服务器公网 IP 了：

![登陆 Chevereto](http://101.200.84.36/images/2022/03/31/202203312107148.png)

当然，你最初看到的不是这个，而是一个初始界面。我没办法演示，但是你知道要填写这类信息就行：

```shell
Database host --> 这个不用动
Database name --> 这个是你刚刚新建的数据库名字，我记得是 picbed
Database user --> 填 root 
Database user password --> 上面你创建的密码，我的是 chunshuyumao
```

之后就是你的一些信息了，随便填写，其中那三个邮箱地址可以一样。最后的 **Website mode** 选择 **Person**，如果你用的是中文的话是**个人**的意思。最后登陆即可。

### 配置

登陆 Chevereto, 右上角选择 Dashbord --> Settings --> Website，修改 Website privacy mode 为 private, 这样你的网站就不会被别人接触到了。

![设置](http://101.200.84.36/images/2022/03/31/202203312120424.png)

![设置](http://101.200.84.36/images/2022/03/31/202203312121054.png)

![修改为Website privacy mode 为 priavte](http://101.200.84.36/images/2022/03/31/202203312123309.png)

![上传的图片](http://101.200.84.36/images/2022/03/31/202203312127809.png)

你点击右上角的上传按钮进行上传——这个应该可以调成中文，我只是不想调而已。

![上传](http://101.200.84.36/images/2022/03/31/202203312129523.png)

图床安装完毕。你的服务器里，图床在 /home/wwwroor/default/images 下边。

### 配合 PicGO

如果你愿意使用 PicGO, 我很乐意告诉你怎么做。

打开 PicGo 查找 Chevereto 插件并安装，配置的时候 Url 写 http://ip/api/1/upload . 例如我的：

> 这里的 IP 指的是你的服务器公网 IP ，不是两个字符。如果你选择了其他端口，请把端口带上：
>
> http://ip:port/api/1/upload
>
> 例如：
>
> http://888.777.666:8080/api/1/upload 就是 IP 为 888.777.666 ，端口为 8080

![搜索 Chevereto 插件](http://101.200.84.36/images/2022/03/31/202203312135945.png)

![配置](http://101.200.84.36/images/2022/03/31/202203312136390.png)

还缺一个 Key. 打开我们的 Chevereto --> Dash Board --> Settings --> Website --> API，把你的 Key 复制一份填写即可。

![Key](http://101.200.84.36/images/2022/03/31/202203312139840.png)

最后你就可以使用 PicGo 开心地上传自己的图床了！

——当然，你得记得在 PicGo 中把 Chevereto 设置为默认图床。

# 后语

这次没有超过 3000 字，又是一次胜利！步骤其实很简单，只是我们不够专业，所以才会觉得难。搭建完之后我试了试，发现图床并没有被公众号屏蔽——妙啊！这样一来，我以后要做的就是使用自己的图床，然后找个时间上传到 GitHub. 因为我的服务器只有一年的时间，到期我也未必续费，所以肯定需要转移自己的图片。一年之后的事一年之后再思考吧，现在为以后打算——以后就没事可做了。