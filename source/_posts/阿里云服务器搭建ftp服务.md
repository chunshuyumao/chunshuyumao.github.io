---
title: 阿里云服务器搭建 FTP 服务
tags:
  - FTP
categories:
  - 服务器
math: false
mermaid: false
date: 2022-03-04 09:06:03
index_img:
excerpt: 最近在阿里云购买了一个云服务器，ECS共享型4n。恰好拿来练手，所以搭建了一个ftp服务器。其中还是遇到了很多问题，但是后来都慢慢解决了，故此记录一下。
---

# FTP协议

FTP 协议，全称 文件传输协议(File Transfer Protocol) ，是简单、高速的网络文件传输协议，支持 TCP/IP 。早期的网络共享通常都是使用这个协议，现在也仍然有。当然，在其他的系统上有更好的选择，例如 samba 等，但是在实操方面我只接触过 FTP 。

 FTP 可以搭建家庭内部的文件共享(局域网)和互联网文件共享(公网)。因为自己有了一台云服务器，所以打算在部署自己的 FTP 服务器。如果有想法，可以以后搭建一个家庭的 FTP 它不香吗？

---

## 准备

在开始之前，我们先准备需要的东西：

1. 一台云服务器。
2.  SSH 客户端。
3.  FTP 服务端，我使用 vsFTPd ； FTP 客户端，我选择 FTP 。

### 服务器

我用的是阿里云 ECS共享型4n ， Rocky Linux[^1] 系统，兼容 CentOS7/8 和 RHEL 。由于 CentOS6 我没用过，所以不知道区别如何。配置什么的不重要。

如果不使用云服务器而是使用虚拟机的话，那就什么都不重要了。

###  SSH 客户端

使用 SSH 是为了连接服务器，当然，也可以使用服务器官方提供的 VPC 。如果没有这些概念，那就使用 SSH 。
对于虚拟机用户，我还是建议使用 SSH ，因为 VMware 和 Virtual Box 的剪贴板共享功能不大好使，特别是不同操作系统之间互通。

---

# 连接服务器

## 配置安全组规则

在获得自己的服务器之后，到阿里云帐号的 控制台->云服务器ECS->实例->管理->安装组 。如果之前配置过自己的安全组，就在安全组的 配置规则 中选择 快速添加 ，然后设置选择全部， 添加 。这一步很重要，因为我之前在修改 SSH 端口和配置 FTP 时忘了这一步，导致一直在和防火墙进行斗争，最后发现是这里的端口没开放。

![添加规则](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203040956159.png)

## SSH连接

默认情况下，服务器的 SSH 服务端( SSHd )是开放的，如果想查看是否开放，可以使用`sudo systemctl status sshd`查看。老版本使用 init.d 我不熟悉，所以使用 systemctl 。如果你使用的是 CentOS6 ，那只能网上查资料了。

```shell
sudo systemctl status sshd
```

如果打开了，就会出现 active 的提示：

![sshd服务端已打开](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041006651.png)

如果是 inactive ，那就打开 SSHd ，先使用`sudo systemctl enable sshd`设置开机自启动，然后再`start`：

```shell
# enable设置自启动，start启动
sudo systemctl enable sshd 
sudo systemctl start sshd
```

客户端连接使用`ssh username@ip -p portnumber`。其中，如果你修改了 SSH 端口，就加上`-p 端口`，正常情况下可以省略这个参数。`username`是你的服务器用户名，不建议使用 root 登陆，` ip`是你的服务器 公网IP ，注意不是 局域网IP ，通过`ifconfig`和`ip addr`查到的是 局域网IP ， 公网IP 可以在阿里云控制台查找，通常服务器开通之后也会有手机短信告知你的服务器地址。

```shell
ssh chunshuyumao@192.168.0.10 -p 1234
```

连接上之后，输入密码就行了。如果想使用无密码登陆，可以在客户端使用`ssh-keygen`生成公私钥。`-t`是加密的类型，`-C`是注释，也可以不使用`-C`和之后的参数。回车三下，然后到自己的的家目录下就找到`.ssh`文件夹，里面有生成的公私钥。

![生成公私钥](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041022580.png)

然后通过`ssh-copy-id username@ip`传输自己的公钥，不断确定即可。如果有多个公钥，可以使用参数`-i file.pub`指定使用哪一个。这里不过多描述。

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub chunshuyumao@192.168.0.10
```

登陆成功的提示。如果觉得记住自己的 公网IP 困难，可以写一个脚本自动登陆。例如我下边就是写了一个叫做`swc`(switch)的函数，快速切换服务器——如果你使用的是其他 SSH客户端 ，那算我没说。

![登陆成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041031461.png)

---

## 开始配置 FTP

我们花了太多时间在 SSH 的连接上了。

## 下载 FTP 客户端和服务端

服务端和客户端我使用的是`vsftpd`、`ftp`，在服务器中可以两个都下载，用于调试：

```shell
sudo yum install -y vsftpd ftp
```

![安装FTP客户端和服务端](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041043259.png)

当然，你也可以只在自己的电脑下载`ftp`，服务器只下载`vsftpd`。其他系统下载可以网上查教程，或者先用自己的系统查找。例如我的 Manjaro系统 ，但是它没有`ftp`这个单独的软件包，而是自带了。

```shell
pacman -Ss vftpd
pacman -Ss ftp
```

![查找ftp服务端和客户端软件包示例](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041040266.png)



## 配置服务端

输入`sudo vim /etc/vsftpd/vsftpd.conf`，编辑配置，可以参考官网的介绍[^2]。当然，`vim`可以修改成你喜欢的任何编辑器——重点是你的服务器得下载了，例如`nano`。一定要用`sudo`不然你无法修改配置。

```shell
 sudo vim /etc/vsftpd/vsftpd.conf
```

![编辑配置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041045484.png)

修改`anonymous_enable=YES`,这个配置是允许匿名登陆，匿名登陆表示任何人只要知道你的服务器 公网IP 就可以登陆，默认的文件夹保存在`/var/ftp`中。

![image-20220304104811696](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041048745.png)

## 开启FTP服务

首先，查看你的防火墙是否打开。`firewalld`通常是 RedHat 高版本系统( CentOS7/8、RockyLinux )的默认防火墙，`iptables`是 Debian 系列( Ubuntu 等)默认的防火墙。

```shell
sudo systemctl status firewalld
# 或者
sudo systemctl status iptables
```

![防火墙](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041051462.png)

如果是开启( active )的，解决办法有两个： 关闭 和 开放指定端口和服务 。

1. 关闭防火墙。输入命令`sudo systemctl stop firewalld`。关闭自启输入命令`sudo systemctl disable firewalld`。

   ```shell
   sudo systemctl stop firewalld
   ```

2. 开放指定服务和端口。输入`sudo firewall-cmd --add-port=21/tcp --permanent `和`sudo firewall-cmd --add-service=ftp --permanent`。如果想让服务器保持开放这个端口，可以加`--permanent`，否则不用加。然后重新启动防火墙。

   ```shell
   sudo firewall-cmd --add-port=21/tcp --permanent
   sudo firewall-cmd --add-service=ftp --permanent
   sudo firewall-cmd --reload
   ```

   查看防火墙规则:

   ```shell
   sudo firewall-cmd --list-all
   ```

   ![规则](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041104495.png)

开启 vsFTPd 服务和自启动:

```shell
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
```

检查 vsftpd 服务是否打开：

![检查是否打开](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041110173.png)

## 客户端连接

在服务器登陆 FTP 服务，由于是服务器登陆，所以可以使用`localhost`作为 IP 。登陆 匿名用户 ，帐号是`ftp`，密码直接回车。然后就可以使用 FTP 服务了。

```shell
ftp localhost
```

![服务器客户端登陆](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041114988.png)

本地客户端登陆，使用`ftp 公网IP`,一样进行登陆。

```shell
ftp 101.200.84.36
```

![本地客户端登陆](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041117349.png)



至此， FTP 服务搭建完成。下载一个文件试试:

![下载文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203041120475.png)

看到，由于我们使用的是 FTP主动模式 ，所以从服务器下载的时候，服务器也建议使用 被动模式 。不过目前的 主动 和 被动 对我而言没有区别，所以没有开启被动模式。如果不希望别人登陆自己的 FTP ，可以在配置文件中( /etc/vsftpd/vsftpd.conf )把`anonymous_enable`改成`NO`。

---

# 总结

本来搭建 FTP 是很简单的，但是自己的安全组规则没配好，导致一直有问题。我不是很清楚安全组规则和防火墙的区别——所以也许在一些人的操作中，不用控制防火墙也能够使用 FTP 。懒了，所以不想去探究，也许有时间的话我会试试，不过不是现在。

配置 SSH 端口的时候也出现了无法使用指定端口的问题。网上查了一堆博客，发现大家基本是在调试 SELinux 和 防火墙 。一个也解决不了问题，最后还是通过阿里云控制台修改了安全组规则，所以这个也很迷。我的服务器 SELinux 默认是关闭的，实际上开启的话，配置 SSH 端口更麻烦，但是自然也更安全。

---

##相关网址

[^1]: https://rockylinux.org
[^2]: http://vsftpd.beasts.org/vsftpd_conf.html
