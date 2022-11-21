---
title: FTP服务进阶
hide: false
tags:
  - FTP
categories:
  - 服务器
math: false
mermaid: false
date: 2022-03-12 09:42:00
index_img:
excerpt: 初步搭建FTP服务之后更进一步的学习。
---

# 1. 前言

上次搭建的 FTP 服务满足基本的要求，但是还有一些情况我们没有考虑。这次主要针对两种情况进行优化：

1. <ruby>被动模式<rt>passive mode</rt></ruby>(PASV)。
2. <ruby>虚拟用户<rt>virtual users</rt></ruby>。

# 2. 被动模式

 2.1. FTP 服务的两种模式

FTP 服务端和客户端的通信有两种模式，一个是<ruby>主动<rt>PORT</rt></ruby>模式，另一种是<ruby>被动<rt>PASV</rt></ruby>模式。两种模式的主要区别是相对于服务端而言。

1. 主动模式。客户端通过随机端口 X 连接到服务器 21 端口，告诉服务器：用你的 20 端口连接我的 Y 端口。随后 Y 端口和 20 端口进行数据传输。
2. 被动模式。客户端通过随机端口 X 连接到服务器 21 端口，服务器告诉客户端：连接到我的 Z 端口。随后客户端通过自己的另一个随机端口 W 连接到服务器的 Z 端口。

这样看来，被动模式似乎更麻烦，实际确实如此，但这种麻烦是有原因的。大多数人的电脑使用的是 Windows 系统， Windows 系统的防火墙一般默认开启。防火墙通常会阻止各种端口的接入。主动模式下，服务器用 20 端口连接客户端的 Y 端口，如果防火墙开启，就会阻止 20 端口和 Y 端口连接。下面演示一下：

首先关闭我的防火墙，也就是没有防火墙干扰的情况下的连接。下面查看防火墙状态之后发现是<ruby>未激活的<rt>inactive</rt></ruby>。

![没有防火墙](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121023625.png)

没有防火墙的时候，可以正常使用 ls 命令列出登陆目录下的文件。

接下来打开防火墙再次进行连接：

![激活防火墙之后连接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121026609.png)

这次防火墙处于激活状态，我们可以正常登陆 FTP 服务端，但是无法发送指令。因为登陆使用的随机端口 X 可以被服务器接收，服务器收到连接 Y 端口的指令并用 20 端口连接到客户端的 Y 端口的时候被防火墙挡住了。客户端再次发送指令，服务器的所有反馈都没有办法进入客户端。这样，服务器和客户端也就无所谓连接。

想要继续连接有两个办法：

1. 关闭防火墙。
2. 服务端开启<ruby>被动<rt>PASV</rt></ruby>模式。

方法一简单粗暴，但是有些人不希望自己的防火墙关闭，为了一劳永逸只能选择开启被动模式。

服务端开启被动模式。我们查看防火墙一直开着，登陆 FTP 使用主动模式的指令还是失败，使用 `passive`进入被动模式，然后输入指令，发现可以正常操作。

![开启被动模式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121038954.png)

这样，被动模式就成功在客户端存在防火墙的情况下仍然可以和服务端连接。

> 默认情况下防火墙只会阻止某些端口的进入，客户端端口可以正常向外访问，所以被动模式的服务器给了客户端一个端口，让客户端去连接服务器，主动模式下则是服务器连接客户端。因此，主动和被动是相对服务器而言的：主动模式下服务器连接，被动模式下客户端连接。

 2.2. 开启被动模式

连接服务器，使用你喜欢的编辑器打开 vsFTPd 配置文件

```shell
sudo vim /etc/vsftpd/vsftpd.conf
```

直接转到最后一行，添加以下配置

```shell
pasv_enable=YES
pasv_addr_resolve=YES
pasv_address=101.200.84.36
pasv_min_port=41001
pasv_max_port=42010
```

![修改配置文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121049175.png)

解释一下，`pasv_enable` 配置<ruby>被动模式 <rt>passive</rt></ruby>( PASV) 是否开启。默认情况下被动模式是开启的，但是不起作用。

`pasv_addr_resolve` 配置的是域名解析。虽然不一定用到，但是还是写下吧。

`pasv_address` 配置返回的 IP 。如果你的 vsFTPd 服务器是安装在局域网，例如自己家庭使用、虚拟机练习，那这个 IP 就填你的局域网 IP ；如果 vsFTPd 部署在自己服务器，那就使用服务器公网 IP 。

> 公网 IP 是你购买服务器后，云服务商手机告诉你的公网 IP。如果忘了，可以到服务商提供的控制台查看。
>
> 局域网 IP 简单，就是你使用  `ip addr` 或者 `ifconfig`看到的 IP 。

如果不填写 `pasv_address` 而且你的服务器是云服务器，即使你的被动模式开启了，其他人也连接不上，会出现这样的提示：

![取消公网 IP ](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121100091.png)

![没有配置公网 IP ](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121059035.png)

因为被动模式开启但没有配置公网 IP ，服务器就使用局域网 IP 作为被动模式的 IP，例如我服务器的局域网 IP  `172.27.163.6 `。因此使用云服务器一定要改 `pasv_address`。

`pasv_min_port` 和 `pasv_max_port` 是服务器开放的端口范围，什么意思？就是服务器可能也要开防火墙——总不能让它冒险吧——因此保留这些端口不挡住，可以给客户端在被动模式时连接。范围可大可小——但是注意，由于一个端口一般只能连接一台客户端，如果 FTP 服务器使用人数比较多可以把端口范围放大，否则可以调小。建议使用高端口，但是也不要超过五位数。

设置完成之后保存退出，重启 vsFTPd 服务

```shell
sudo systemctl restart vsftpd
```

到这里，被动模式已经开启。

![再次测试](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121111379.png)

一般来说，被动模式和主动模式可以同时开启：以备不时之需。

Windows 的 FTP 客户端一般都会选择被动模式登陆，如果服务端没有开启被动模式，客户端就无法获取服务端的文件。这种时候可以查看客户端的设置，通常可以选择连接模式。如果是 Linux ,一般都是手动操作。保不准自己进入了被动模式，那怎么退出？大多数的 FTP 客户端会提供一些指令，可以使用 `help` 或者 `?`进行查询：

![FTP 客户端命令](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121115309.png)

这么来说，我的客户端退出被动模式就很简单了:

```shell
passive off
```

![退出被动模式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121117065.png)

# 3. 虚拟用户

刚搭建的 FTP 服务器开启了匿名模式，即可以使用 ftp 作为用户名无密码登陆；还允许本地用户登陆，即允许服务器上的用户登陆。这样有一定风险的：本地用户拥有的权限，登陆后仍然拥有。比如我进行登陆，可以直接看到整个服务器的目录结构：

![服务器目录结构](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121123862.png)

这就产生了权限漏洞。如果我们即想要分享，又不希望别人对自己的文件进行不必要的干扰，最好的办法就是创建虚拟用户，然后进行权限限制。

 3.1. 添加服务器用户

首先我们需要在服务器创建一个映射虚拟用户的不可登陆的用户帐号：

```shell
sudo useradd vuser -d /var/ftp -s /sbin/nologin
```

解释一下，`-d` 是指定虚拟用户的家目录，我们之前已经创建过这个目录作为匿名用户的家目录，现在我们不打算使用匿名帐户，直接把家目录划给虚拟用户。

虚拟用户只是一个系统权限映射的用户，没有必要让它登陆服务器，所以设置使用 `-s` 指定它不可登陆。

> `-s`一般是创建用户时指定使用的 `shell` ， Linux 中通过把 `shell` 指定到 `nologin` 实现禁止登陆

创建之后可以使用

```shell
cat /etc/passwd | grep vuser
```

查看用户信息

![查看虚拟用户信息](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121143056.png)

 3.2. 添加虚拟用户

进入 `/etc/vsftpd` 添加一个文件，我这里是 `vuser.cfg` ，在里边填入虚拟用户的信息。注意，单行是用户名，双行是用户密码：

![添加虚拟用户](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121148543.png)

```shell
lileilei
20220202
hanmeimei
20220303
```

保存退出。

`cfg`文件不能被 vsFTPd 读取，需要转化格式，变成可读的 `db` 格式：

```shell
sudo db_load -T -t hash -f /etc/vsftpd/vuser.cfg /etc/vsftpd/vuser.db
```

![转化为 db 格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121153783.png)

把 `db` 改为只有 root 用户可以读写

```shell
sudo chmod 600 vuser.db
```

![修改权限](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121253782.png)

让 vsFTPd 读取自己配置的 `db` ，需要配置<ruby>可插入认证模块<rt>Pluggable Authentication Modules</rt></ruby>( PAM )文件。

创建 PAM 文件：

```shell
sudo vim /etc/pam.d/vsftpd.vuser
```

输入

```shell
auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser
account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser
```

`db` 之后的配置文件是我们上面写的 vuser.db 文件。

 3.3. 修改 vsFTPd 配置文件

进入 `/etc/vsftpd` 目录，如果没有 `vuserconfig`目录，则创建一个新的目录。

```shell
cd /etc/vsftpd/
sudo mkdir vuserconfig
```

然后进入 vuserconfig 目录，目录里边是我们的虚拟用户的权限文件。想要限制谁的权限，就在这个目录创建同名的文件。我配置一个用户的权限：

```shell
sudo vim lileilei
```

配置权限

```shell
local_root=/var/ftp
write_enable=YES
anon_umask=022
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
download_enable=YES
```

修改 /etc/vsftpd/vsftpd.conf 文件：注释掉 pam_service_name=vsftpd，这是默认的用户配置文件，我们添加了虚拟用户，所以改成我们自己的 vsftpd.vuser ，这里的 guest_username=vuser 的 vuser 与我们在服务器建立的 vuser 用户名字一样。

```shell
guest_enable=YES
guest_username=vuser
user_config_dir=/etc/vsftpd/vuserconfig
allow_writeable_chroot=YES
pam_service_name=vsftpd.vuser
```

![修改 vsFTPd 配置文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121327337.png)

最后重启 vsFTPd 服务

```shell
sudo systemctl restart vsftpd
```

客户端尝试本地用户登陆，发现失败了；使用虚拟用户 lileilei 登陆成功。

![尝试本地用户登陆](https://cdn.jsdelivr.net/gh/chunshuyumao/202203bf@master/202203121332066.png)

虚拟用户，创建完毕！如果要继续添加新用户，可以往 vuser.cfg 添加，然后生成 `db` 文件就可以了。

# 4. 后语

上面就是我们 FTP 的进阶配置。可以看到，虽然 FTP 显得很古老，但是仍然具有很强大的功能。通过一些配置可以实现虚拟用户登陆、配置登陆用户的权限等等，所以 FTP 仍然可以在现在这个互联网时代保持其优异的分享功能。

实际上，如果是在家庭中，我们可以自己搭建一个局域网内的 FTP 服务器，让自己的家庭成员可以分享自己的文件，这比使用公有云更安全，对隐私的保护更出色。

FTP 可探索的配置还是很多的，所以好好探索吧！
