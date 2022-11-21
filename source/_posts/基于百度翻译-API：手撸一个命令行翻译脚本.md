---
title: 基于百度翻译 API：手撸一个命令行翻译脚本
hide: false
tags:
  - 翻译
categories:
  - Linux
math: false
mermaid: false
date: 2022-03-24 20:54:31
index_img:
excerpt: 一件乐事和伤心事
---

# 前言

今 天费了九牛二虎之力，手撸了命令行翻译软件。原本是件好事，晚上准备记录的时候遇见了一件伤心事：我的 Gitee  图床仓库被屏蔽了。不大清楚为什么自己的图片外链访问过多？想想 PicGO 上是使用 Gitee  的不在少数，偏偏我遇上了。看来自己的博客每次都有成千上万人浏览。不过我本就没有准备在一棵树上吊死，我还有 sm.ms  图床，虽然慢了点。卑躬屈膝地给 Gitee 管理员写了封信，好歹让我能备份自己的图片吧，咱也没啥要求了。既然如此，大家还是好好考虑一下要不要用  Gitee，至少目前我是没办法用了。

闲话少叙，正事还没讲呢。

一直以来我都是用 translate-shell[^1]，一个简单的命令行翻译软件，给自己翻译。最近想想，我好像之前申请过百度翻译开放平台的一个账号。那时候我还用 Windows，所以也使用**知云文献翻译**[^2]。**知云文献翻译**还是很良心的，可惜没有 Linux 平台。**知云**允许自定义<ruby>百度翻译API<rt>Application Programming Interface</rt></ruby>，好奇心驱使下我就申请了百度翻译的账号。后来用了多少没看，文献也啥好翻译的，时不时查一个词还不错。

translate-shell[^3] 功能强大，如果使用 Linux，可以直接命令行安装，我用的 Manjaro Linux, 所以安装方式是：

```shell
sudo pacman -S translate-shell
```

其他发行版就用你们熟悉的方式吧。安装完成之后使用 `trans` 进行翻译。具体可以参照<ruby>帮助<rt>man</rt></ruby>。

使用一阵子之后，我突然也想搞自己的翻译软件？当然，我打算用 shell/bash 直接写个脚本。其实这样也有好处：好歹让我习惯 bash 的用法。话不多少，准备：

1. 百度翻译开发账号[^4]
2. Linux 平台，可以是<ruby>适用于 Windows 的 Linux 子系统( WSL )<rt>Windows Subversion for Linux</rt></ruby>，Linux 虚拟机、主机、服务器，Cygwin[^5]，也许 Windows 上的 Git Bash 也可以？

我使用的是 Manjaro 5.15 内核，bash 是 5.1。其实这些都不用在意：我列出来就是假装自己很懂而已，大多数情况下这些东东都是兼容的。

# 申请百度翻译账号

开工之前，我们先去百度翻译官网申请一个账号：

![申请账号](https://cdn.jsdelivr.net/gh/chunshuyumao/202203/202203250946067.png)

申请完之后选择**文档与支持–>开发者文档**，选择右边的**开通链接**，选择**通用翻译**。其他的服务以后有兴趣自己试试。

![开通链接](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251000872.png)

填申请。因为我已经申请过通用翻译，所以这里开了一个其他翻译方式，对照着看就差不多了。填完之后直接申请，基本秒通过。

![申请](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251002303.png)

![通过](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251004510.png)

完成之后选择**开发者信息**，记住右边的 **APP ID** 和密<ruby>钥<rt>yuè</rt></ruby>。

![重要东东](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251028435.png)

选择**文档与支持**，**接入服务**版块介绍了**通用翻译**的一些信息，基础的翻译而已，不用太较真——**开发者文档**有**通用翻译**使用的介绍。接下来我们基本都是看**通用翻译**的**开发文档**。很大程度上，大家可以自己动手造，用不着看我的演示。不过我还是介绍自己踩过的坑。百度的开发者文档做的还挺好，后面还有各种语言的事例，如果不使用 Bash 的话可以看看——我之前也用 Python 写过一个，觉得还可以。

![文档支持](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251029605.png)

![通用翻译](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251031898.png)

![使用方式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251032376.png)

# 开始制作

我后面基本是按照百度翻译的文档进行介绍，所以难免有抄袭的铁证。

首先是**通用翻译**的地址

```shell
https://fanyi-api.baidu.com/api/trans/vip/translate
```

随便打开一个 Shell 脚本文件。我用 VIM 编辑器，所以使用 VIM 打开。

```shell
vim mt.sh
```

> 因为 Bash 兼容性很强，现在的 Linux 基本都是配置 Bash，所以我可能会 Shell 和 Bash 混用，大家只要知道我讲的是 Bash 就行了。
>
> Bash 是 Bourne Again Shell 的缩写，以前有个 Bourne Shell, 所以是 Again .
>
> 文件名叫做 mt.sh 是因为它是 My Translater，懒得写那么多字母，干脆就是 mt.

在脚本最上面写下

```shell
#!/bin/bash
# encoding:utf-8
```

​	![杀棒(shabang)](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251048397.png)

> `#！` 这个符号读作 shabang 或者 hashbang. 它的作用不用深究，例行公事写下来就行。
>
> 下面的 `encoding` 相信写过 Python 的人都比较习惯，例行公事，不写也没事。

接下来就是写下自己的信息，HTTPS 地址和 APPID、密钥等。`declare -r` 声明这些是常量( Read Only )，可有可无，例行公事。文档中还提到 Salt 参数，说是一个随机码，既然是随机的，我们也随便给它一个。

```shell
# 翻译 HTTPS 地址
declare -r basic_url="https://fanyi-api.baidu.com/api/trans/vip/translate"
# APPID
declare -r appid="20222231001673136"
# 密钥
declare -r token="L4Jr77zOjy91cKx9_4oP"
```

![信息](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251053579.png)

![Salt](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251055525.png)

在 Linux 系统生成随机数的方式还是蛮多，我对 `openssl rand` 比较满意，所以就用它了，在自己的脚本中写入:

```shell
declare -r salt=$(openssl rand -base64 18|cut -c 10)
```

1. 先看内部， `openssl rand` 是一条指令，用于生成随机数，`-base64` 是使用 64 个可打印的字符，`18` 表示我要生成 18 个字符。
2. `|` 在 Linux 中表示**管道符**，意思是把右边输出当作左边的输入。也就是说，**管道符**右边的输出是左边的输入。不了解也没事，知道这么一件事就可以了。
3. `cut` 表示截断输入，通常是字符或者文件。`-c` 表示按照字符截断，`10` 表示截取第 10 个字符。如果想截取 10 个字符，需要用 `1-10`。原本我是想截取 10 个的，但是发现的时候已经截图了，那就算了。
4. `$()` 表示获取括号里边计算的结果。简而言之就是，获取括号里指令的结果。

上面的 `cut -c 10` 指令可有可无，一来文档没有说 Salt 有没有长度限制，二来其实管道符右边的式子也可以控制生成的长度。随便吧，最简单就是以下的表示：

```shell
declare -r salt=$(openssl rand -base64 10)
```

## 签名

接下来准备生成<ruby>签名<rt>sign</rt></ruby>。什么是<ruby>签名<rt>sign</rt></ruby>？其实就是我们翻译的请求信息。根据文档要求，签名应该长这样:

```shell
MD5(appid+q+salt+token)
```

其中，`q` 是 query 的缩写，就是我们想要翻译的文字，UTF-8 编码——不了解编码不用管，知道就行。

我们准备了 appid、salt 和 token 了，现在只差 q。为了方便调试，我们声明一个 q, 取名 query。

```shell
delcare query="like"
```

![query](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251121224.png)

首先拼接字符串，然后取 MD5 

```
tmpStr="$appid$query$salt$token"
sign=$(echo -n tmpStr | md5sum | cut -c 32)
```

解释一下，我们用 `md5sum` 生成 MD5，然后只取 32 个字符——因为文档说我们只需要 32 位。Shell 语言和 Perl 语言很相似，可能在其他高级语言里想要获取某一个变量的值，我们可以直接使用这个变量名；但是在 Shell 里，我们需要用 `$` 符号获取变量值，最正规的方式是：`${appid}`。因为要拼接成一个字符串，所以建议拼接的两边加上双引号。Shell 的字符串拼接也很有趣，直接把两个字符串放一起就行了。

> `-n` 非常重要，别丢了

举个例子，其他语言的语言拼接：

```shell
# python
str1 = "Hello"
str2 = "World"
str3 = str1 + str2

# cpp
std::string str1{ "Hello" };
std::string str2{ "World" };
std::string str3{ str1 + str2 };

# lua
local str1 = "Hello"
local str2 = "World"
local str3 = str1..str2

# Shell/Perl
str1="Hello"
str2="World"
str3="$str1$str2"
```

闲话少叙，其实上面的两行可以写成一行：

```shell
sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 32)
```

## 翻译

获取签名之后就可以使用百度翻译提供的 API 翻译了。在 Shell 脚本中，我们使用大名鼎鼎的 cURL 进行网路请求。首先声明，我对 cURL 不熟，一般我都不用它——因为学不会。

cURL 请求的命令是：

```shell
curl url
```

所以我们需要提供一个 url. 上面我们准备了百度翻译的 API, 现在开始拼接。在脚本中输入：

```shell
url="$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign"
curl "$url"
```

或者直接一行

```shell
curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign"
```

如果使用 VIM，那就 <kbd>Esc</kbd>，然后输入 `:wq`，回车退出。在命令行上输入

```shell
chmod u+x mt.sh
```

![给予可执行权限](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251220209.png)

输入 `./mt.sh` 执行脚本。可以看到有返回了，结果就在 `dst` 里边。

![结果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251255943.png)

现在要做的就是取出我们的结果，使用 `-e` 进行转义输入：

```shell
result=$(curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign")
echo -e "$result"

# 或者直接一行
echo -e "$(curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign")"
```

![转义](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251258949.png)

## 字符串截断

在获取结果之前，我们先说说如何在 Shell 中截取字符串。就拿我们上面的字符串做例子。现在我们要做的是把“喜欢”取出来。使用下面的命令：

```shell
result="${result##*dst\":\")"
echo -e ${result%\"\}\]\}}
```

![获取结果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251308513.png)

解释一下：

1. 首先，分割字符串使用 `#` 和 `%` ，前者表示从左往右删除，后者表示从右往左删除。举个例子 `str='chunshuyumao'` 如果我想删掉 `chunshu`，就用 `${str#chunshu}`。如果我懒了，不想写那么多个字母，那就写成这样 `${str#*shu}`，用 `*` 表示任意字符，这样只要找到了 `shu`，就把 `shu` 和它之前的字符全删掉。同理，`%` 表示从右边开始算起。那两个 `#` 表示什么？在 Linux 中有个习惯，两个东西通常表示极端——例如在 VIM 中，`d` 表示删除，`dd` 就直接删除一行；`y` 表示复制，`yy` 就直接复制一行。`#` 表示从左边找的第一个符合条件的字符串，两个 `#` 就是从左边找的最后一个符合条件的字符串。举个例子

   `str="chunshuyumaoshuzhuang"`，如果用 `${str#*shu}` 就会得到 `yumaoshuzhuang` ,使用 `${str##*shu}` 就会得到 `zhuang`。同理，`%%` 表示从右边数最后一个匹配的字符串。

2. 因为引号( “ ) 是特殊的字符串，所以我们用 `\` 转义，所以第一行的原本想法是 `${result#*dst":"}`，加上转义字符之后就是 `${result##*dst\":\"}`。

3. 括号也是特殊字符，所以一并转义

这样就得到结果了。不过大家可能也看到下面的输出了，我们不希望有这个输出。

```shell
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    76  100    76    0     0    301      0 --:--:-- --:--:-- --:--:--   301
```

在脚本使用 cURL 的后面加上 `2>/dev/null` 这个语句是把一些不重要的输出放到 `/dev/null` 这个文件。不用担心，这个文件没啥用。参看下面的结果：

```shell
result=$(curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
```

![重定向输出](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251326986.png)

![最后的结果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251329558.png)



截至目前，你的脚本看起来是这样的：

```shell
#!/bin/bash
# encoding:utf-8

# 翻译 HTTPS 地址
declare -r basic_url="https://api.fanyi.baidu.com/api/trans/vip/translate"
# 你的 APPID
declare -r appid="xxxxxxxxxxxxxxxxxxxx"
# 你的密钥
declare -r token="xxxxxxxxxxxxxxxxxxxx"
# random code 
declare salt=$(openssl rand -base64 18|cut -c 1-10)

declare query="like"

sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)
result=$(curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
result="${result##*dst\":\"}"
echo -e ${result%\"\}\]\}}
```

既然已经成功了，我们开始修改 query 的内容。

## 输入

第一个要解决的就是 query 的输入问题。我们希望这样使用自己的翻译器 `./mt.sh "English"`.  Shell 脚本和其他语言一样，可以获取它的命令行参数。听不懂没事，假设我这样调用自己的脚本：

```shell
./mt.sh chun shu yu mao
```

`./mt.sh`、`chun`、`shu`、`yu`、`mao` 就是命令行参数。脚本中使用 `$n` 的方式获取参数，其中 `n` 表示阿拉伯数字。我下面输出一下参数

![输出参数](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251338985.png)

![输出参数](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251338982.png)

知道这个之后，我们修改一下自己的脚本：

```shell
declare query="$1"
```

![桉树输入](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251419011.png)

单词可以输入了，我们试着输入句子如何？

![句子](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251419770.png)

失败了，很显然没有结果，我们试着打印 `result`。

![打印结果](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251420484.png)

![打印](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251428364.png)

没有任何反馈。其实这是因为我们的句子有问题，看一下文档：

![URL encode](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251428097.png)

我们发送请求的时候需要进行编码。举个例子，上浏览器上搜 `I'm so handsome` 的时候，地址栏显示是 `I'm+so+handsome` . 类似这种就是编码——实际并非完全如此，知道大概就行。文档说使用 URL Encoding，直接使用我给的式子就行：

![URL encode](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251428869.png)

```shell
declare query="$1"

sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)

query=$(echo "$query" | tr -d '\n' | xxd -plain | sed 's/\(..\)/%\1/g')
query=$(echo $query | sed 's/ //g' )

result=$(curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
result="${result##*dst\":\"}"
echo -e ${result%\"\}\]\}}
```

![翻译句子](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251437661.png)

ok, 基本完成了。到这里，脚本实现翻译已经完成了。如果你觉得这已经不错了，那就可以结束这篇文章了。如果觉得还想要改进，那就继续阅读。

## 扩展

我们用脚本和百度翻译 API 完成了自己的第一个翻译脚本。接下来进行多语言拓展。首先我们看一下百度支持的常用翻译：

![支持的翻译](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251441719.png)

翻译的语言信息要放哪里呢？来看看我们之前写的请求地址：

```shell
result=$(curl "$basic_url?q=$query&from=en&to=zh&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
```

上面我们从哪种语言翻译过去，就使用 `from=代号`, 翻译成哪种语言就用 `to=代号`。好的，我们改改，改成翻译为文言文：

```shell
result=$(curl "$basic_url?q=$query&from=en&to=wyw&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
```

![文言文翻译](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251447688.png)

为了让使用者可以控制翻译的设置，我们可以添加一些参数，例如我已经做好的脚本：

![help](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251500470.png)

![列出支持的语言](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251509696.png)

![中英翻译](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203251513256.png)

所以接下来，我们就要实现上面这些功能——不过不是现在。

# 相关网址

[^1]: https://github.com/soimort/translate-shell/
[^2]: http://www.zhiyunwenxian.cn/
[^3]: https://sourceforge.net/projects/translate-shell.mirror/
[^4]: https://fanyi-api.baidu.com/ 
[^5]: https://www.cygwin.com/