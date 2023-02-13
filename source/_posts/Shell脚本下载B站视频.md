---
title: Shell脚本下载B站视频
hide: false
tags:
  - 脚本
categories:
  - Linux
math: false
mermaid: false
date: 2022-06-08 21:55:27
index_img:
excerpt: Shell 脚本下载 B站 视频
---

# 前言

前一阵子想在 B站 下载点东西，可没有登陆各种网站的习惯。网上存在一些在线解析 B站 视频的网站提供下载，不过我当时需要的是批量下载——可能是有洁癖吧，什么东西都觉得本地保留一份才是最安全的。这样一来，线上解析的网站就难找了。再说了，有些网站为了维持运营，不得不添加一些广告，这就很烦人了——好吧，白嫖没理由抱怨。

后来不得不使用了 [You-get](https://github.com/soimort/you-get) 这个堪称下载神器的东西。You-get 虽然可以实现批量下载，但是问题也很明显，就是批量下载可能会卡住，一卡住就得从来，见鬼。为了加快下载速度，自己又得在 You-get 的基础上写一个多线程下载。You-get 用 Python，我一来对这门语言不感兴趣，二来这东东好像后面还是调用了 [FFmpeg](https://ffmpeg.org) 进行音视频合并，这种东东使用简单的 Shell 也可以实现，何必多此一举？

有了想法开始动手实现自己的 B站 视频下载脚本。这里要说说，You-get 的功能太强大，一般我用不上，所以自己的脚本自然看起来简单多了，但是和 You-get 比还是功能太少。自己写的脚本不支持列表下载，不过可以配合自己之前写的多线程实现下载——适合自己的就是最好的。

# 准备

使用 Shell 脚本自然不可能是 Windows 系统，所以 Windows 系统的同志可以散了。想要自己写写的可以随便上网搜搜 Python 语言的，运气好可能搜到一篇真的在教你怎么用 Python 下载的。一般而言，使用国内的搜索引擎会非常痛苦——你不会找到自己想要的东西，你只会找到别人开钱给你看的东西。下面是基本工具：

1. Unix/Linux 系统，也就是除了 Windows 外的系统。
2. [FFmpeg](https://ffmpeg.org/) 一个跨平台的音视频框架。其实就一句话，合并音视频用的。
3. Shell，形形色色的 Shell 都行，为了最大的兼容性，一般是 Bash。
4. JQ，一个 Shell 上处理 [JSON](https://www.json.org) 数据的工具。通常 Unix/Linux 系统不会自带，是用包管理器直接可以下载。

除此之外还使用到 SED、[cURL](https://curl.se) 等工具，一般 Unix/Linux 系统都自带了，所以不单独列出。

# 实现

接下来会分步骤进行实现。

## 构建框架

为了精简脚本的功能，我的想法是脚本如此调用：

```bash
bl.sh https://www.bilibili.com/video/BV1UE411d7QZ
```
也就是 `脚本 + URL`。然后就没有任何功能了——要的就是精简。

首先要创建一个脚本，我用的是 VIM 编辑器，所以下面是基本操作：

```bash
$ vim bl.sh
#!/bin/bash
set -euo pipefail

function main() {


}

main "$@"
```

在上面的 `bl.sh` 脚本中写了一个 `main` 函数作为主函数，并在最后调用这个函数。因为自己最开始接触到的语言是 C/C++ ，总觉得无论如何还是写一个主函数逻辑上才清楚，不喜欢一上来就是各种奇葩语句和乱入的函数堆叠。

`set -euo pipefail` 设置脚本的一些功能，其中 `-e` 表示遇到报错直接退出（EXIT），`-u` 表示使用没有声明的变量（undecleared variable）就直接退出，`-o pipefail` 表示使用管道符失败就报错退出。

设置上面的功能源于我的一些理念：只要没有得到正确执行就是错误，错误没有必要存在，没必要存在就没必要执行，程序直接算失败。这是一种习惯，有些人就喜欢和警告（warning、info）、错误（error）共存，咱也没办法。

为了进行检验，下面会以 `https://www.bilibili.com/video/BV1Sg41137WR` 作为下载事例。

## 使用 cURL

我们先用 cURL 请求目标网址试试：

```bash
$ curl -O "https://www.bilibili.com/video/BV1Sg41137WR"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 47405    0 47405    0     0  83409      0 --:--:-- --:--:-- --:--:-- 83312
$ ls
BV1Sg41137WR
$ file BV1Sg41137WR
BV1Sg41137WR: gzip compressed data, from Unix, original size modulo 2^32 259418
```

![尝试请求](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/08/202206082248087.png "尝试请求")

`curl -O url` 表示把请求网址的结果保存，我们看到，保存之后生成了一个 `BV1Sg41137WR` 的 gZIP 压缩文件。解压这个文件试试：

```bash
$ mv BV1Sg41137WR{,.gz}
$ gzip -d BV1Sg41137WR.gz
$ file BV1Sg41137WR
BV1Sg41137WR: HTML document, Unicode text, UTF-8 text, with very long lines (63971), with no line terminators
```

![解压文件](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/08/202206082249382.png "解压文件")

可以看到，解压之后这个文件其实是一个 HTML 文件，也就是网页的源码。我们要找的视频网址等内容都在里边，接下来就是我们解读了。解释解释上面的行为。

`mv` 是重命名的意思，因为后面需要解压，自然就需要重名了。FreeBSD 等 Unix 直系后代系统习惯通过文件内容判断文件格式。比如上面的 `BV1Sg41137WR` 文件，通过判断可以得知是 gZIP 文件。在 FreeBSD 中可以直接使用 `gzip -d` 解压，因为通过内容可以轻松辨认出来；但是 Linux 系统通过文件扩展名辨认文件， `BV1Sg41137WR` 没有文件名，在 Linux 上使用 `gzip -d` 它就分辨不出来这个文件是啥，自然就拒绝解压。

![错误案例](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/08/202206082255543.png "错误案例")

`mv` 是 `move` 的缩写，用于移动和重命名。按理来说我们的操作应该是

```bash
mv BV1Sg41137WR BV1Sg41137WR.gz
```

`.gz` 是 gZIP 文件的扩展名。可是我们看到上面的操作重复的字符太多了，懒人就不该写那么多的字母，所以 Linux Shell 又有这种操作

```bash
mv BV1Sg41137WR{,.gz}
```

通过在花括号和一个逗号省略相同的部分。于是就有我们的上面的操作。

`gzip -d` 中的 `-d` 是 `--decompress` 的缩写，也就是解压咯。

接下来可以使用文本编辑器查看下载的 HTML 源码，其实最好的方式是通过浏览器直接查看源码。因为已经搞清楚了 B站 尿性，所以我这里就不解释如何通过分析源码了解我们想要的内容。直接告诉大家：我们需要的部分在源码中的 `window.__playinfo__` 部分，可以通过浏览器查看源码然后查找得到。

![目标](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/09/202206091212163.png "目标")

接下来要做的就是把 `__playinfo__=` 后面的内容提取出来，然后通过 jq 格式化显示出来。

## 获取目标内容

已经有了 HTML 源码，现在就是从源码中提取我们需要的部分。上面已经说过，`__playinfo__=` 后面就是我们的目标内容。在命令行上输入

```bash
$ cat BV1Sg41137WR | sed -nr 's/.*__playinfo__=(.*)<\/script><script>.*/\1/gp'
```

![获取需要的部分](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/09/202206091254062.png "获取需要的部分")

这里解释一下，我们需要使用 `BV1Sg41137WR` 的内容，所以通过 `cat` 和管道符 `|` 把内容传递给 SED。SED 参数 `-n` 表示不打印，SED 一般匹配之后会直接打印出来，我们希望打不打印由自己控制，所以默认直接不打印；`-r` 表示后面我们会用到正则表达式（Regular Expression）。

`s/.*__playinfo__=(.*)<\/script><script>.*/\1/gp` 是传递给 SED 的正则匹配模式。在 SED 中使用替换的语法是 `s/old/new/gp`，把 `old` 替换为 `new`，`g` 表示全局替换，SED 默认只替换一次，使用全局替换就是见到的就替换；`p` 表示打印出来，也就是替换之后打印出来；这里的 `gp`是可选的，也就是可有可无。正则中的 `.*` 表示任何字符，`(.*)` 表示任意字符，并把它们分成一个小组，小组的编号从 1 开始，使用 `\1` 代替。如果看不懂就算了，这就是提取内容罢了。

获取内容之后，我们可以看到内容非常乱，简直不是人看的。使用 JQ 进行格式化就好看了：

![格式化之后](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/09/202206091305686.png "格式化之后")

现在我们需要的是视频的链接。B站 由于各种原因——谁都有自己的原因——为了防止别人下载自己的视频，把视频和音频分开存储，这样如果你使用具有嗅探功能的下载器也只能下载到没有音频的视频而已——没办法。

分析我们抓取到内容可以看出，数据中存在视频（video）和音频（audio）两个部分，我们需要做的就是抓取音、视频，然后通过 FFmpeg 合并。

通过格式化后的 JSON 数据可以看到，视频在 `data` 标签下的 `dash` 下的 `video` 下的 `baseUrl` 。我们可以通过 JQ 回去，格式是：

```bash
$ jq '.data.dash.video[0].baseUrl' JSON
```

`video` 加 `[0]` 是因为视频有多种清晰度，我们只需要获取第一个清晰度就行——在编程中，第一个往往从 0 开始数。同理，音频如此获取：

```bash
$ jq '.data.dash.audio[0].baseUrl' JSON
```

JQ 可以一次性获取多个值，所以我们可以简单写成：

```bash
$ jq '.data.dash.video[0].baseUrl, .data.dash.audio[0].baseUrl' JSON
```

最后我们这样获取想要的数据：

```bash
$ cat BV1Sg41137WR | sed -nr 's/.*__playinfo__=(.*)<\/script><script>.*/\1/gp' | jq '.data.dash.video[0].baseUrl, .data.dash.audio[0].baseUrl'

"https://upos-sz-mirrorhw.bilivideo.com/upgcxcode/53/45/348384553/348384553_nb2-1-30080.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1654755278&gen=playurlv2&os=hwbv&oi=3723425079&trid=9f2d5b2f02c543e4919e6644832471cbu&mid=0&platform=pc&upsig=dfcbb72afadc553a5c83c23fcacb7148&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&bvc=vod&nettype=0&orderid=0,3&agrr=1&bw=255483&logo=80000000"
"https://upos-sz-mirrorhw.bilivideo.com/upgcxcode/53/45/348384553/348384553_nb2-1-30280.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1654755278&gen=playurlv2&os=hwbv&oi=3723425079&trid=9f2d5b2f02c543e4919e6644832471cbu&mid=0&platform=pc&upsig=8b68ed8743e01fde1dc6ac48136b3bc4&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&bvc=vod&nettype=0&orderid=0,3&agrr=1&bw=23987&logo=80000000"
```

![获取音、视频链接](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/06/09/202206091318587.png "获取音、视频链接")


可以看到，获取的链接两边有引号，我们不需要这些引号，所以使用 SED 去除。方法如下：

```bash
cat BV1Sg41137WR | sed -nr 's/.*__playinfo__=(.*)<\/script><script>.*/\1/gp' | jq '.data.dash.video[0].baseUrl, .data.dash.audio[0].baseUrl' | sed -nr 's/\"(.*)\"/\1/gp'
```

既然已经获得链接，可以把上面的步骤写进脚本了。

```bash
#!/bin/bash
set -euo pipefail

# 测试的 B站 视频链接
declare url='https://www.bilibili.com/video/BV1Sg41137WR'

# 获取内容的正则表达式，用于 sed
declare -r reg_data='s/.*__playinfo__=(.*)<\/script><script>.*/\1/gp'
# 获取音、视频的正则，用于 jq
declare -r reg_video_audio='.data.dash.video[0].baseUrl, .data.dash.audio[0].baseUrl'
# User-Agent 用于伪装成这成浏览器的请求——因为 B站 发现是脚本在下载视频就会阻止你下载。
# 如果你伪装成浏览器，它就以为你使用浏览器，自然不会影响到它的利益，也就不管你下载多少
# 视频了——毕竟你浏览越多，对它越好
declare -r user_agent='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36'

function main() {

  # 获取原始 HTML 源文件，把输出输出到 /dev/null，同时使用 gzip 进行解压，解压的内容放到
  # raw_html 变量中
  local raw_html=$(curl --user-agent "$user_agent" "$url" 2> /dev/null | gzip -d -)
  local va=($(echo "$raw_html" | sed -nr $reg_data | jq "$reg_video_audio" | sed -nr 's/\"(.*)\"/\1/gp'))
}

main "$@"
```

这里，我们不再下载网页源码成单独一个文件，而是处理之后直接保存到一个变量（raw_html）中。最后把音、视频内容放到一个数组 `va` 中。Shell 中的数组使用 `()` 括起来。现在视频地址在 `va[0]`，音频地址在 `va[1]`。

## 下载音、视频

下载音视频的方法比较简单：给 cURL 添加一个 `referer` 然后就可以下载了，语法如下：

```bash
curl --referer "$url" --user-agent "va[0]" --output "视频.mp4"
curl --referer "$url" --user-agent "va[1]" --output "音频.mp4"
```

所以脚本现在是：

```bash
  local va=($(echo "$raw_html" | sed -nr $reg_data | jq "$reg_video_audio" | sed -nr 's/\"(.*)\"/\1/gp'))
  local tempfile=$(mktmpfile) || exit 1
  curl --referer "$url" --user-agent "va[0]" --output "$tmpfile-video.mp4"
  curl --referer "$url" --user-agent "va[1]" --output "$tmpfile-audio.mp4"
}
```

其中 我们创建（mktmpfile）了一个临时文件（temporary file, tmpfile），并确保只有创建成功才执行下面的语句，否则退出（exit 1）。

然后我们把音、视频的文件名改为临时文件的文件名加 `video` 和 `audio` 等字样，这是防止文件名重复，导致多线程下载失败。

接下来就是合并我们的音、视频。不过我们还没有获取视频的标题——这里我直接给出获取视频标题的正则表达式：

```bash
declare -r reg_title='s/.*<h1 title=.*>(.*)<\/h1>.*/\1/gp'

# 获取标题
title="$(echo "$raw_html" | sed -nr "$reg_title")"
```

最后使用 FFmpeg 进行合并：

```bash
ffmpeg -i "$tmpfile-video.mp4" -i "$tmpfile-audio.mp4" -c copy "$title.mp4" -y -loglevel quiet
```

FFmpeg 的参数介绍一下：

1. `-i` 是 `inputfile` 的缩写，代表输入的文件，也就是要合并的文件。
2. `-c` 是 `codec` 的缩写，编码的方式——这里直接使用复制。
3. `-y` 是 `yes` 的缩写，其实就是直接覆盖文件，它会覆盖之前合并的文件，懒人专用。
4. `-loglevel` 是日志打印的等级，这里是 `quiet` ，也就是不打印任何日志。

FFmpeg 不会用没关系，知道这么回事就行——我也不咋地使用这东东。

合并之后，做好把之前的文件删了，所以再写删除脚本执行过程中产生的文件。

```bash
/bin/rm -r $tmpfile $tmpfile-audio.mp4 $tmpfile-video.mp4
```

所以我们的脚本最现在是这样的：

```bash是
#!/bin/bash
set -euo pipefail

# 获取内容的正则表达式，用于 sed
declare -r reg_data='s/.*__playinfo__=(.*)<\/script><script>.*/\1/gp'
# 获取音、视频的正则，用于 jq
declare -r reg_video_audio='.data.dash.video[0].baseUrl, .data.dash.audio[0].baseUrl'

# 获取标题
declare -r reg_title='s/.*<h1 title=.*>(.*)<\/h1>.*/\1/gp'

# User-Agent 用于伪装成这成浏览器的请求——因为 B站 发现是脚本在下载视频就会阻止你下载。
# 如果你伪装成浏览器，它就以为你使用浏览器，自然不会影响到它的利益，也就不管你下载多少
# 视频了——毕竟你浏览越多，对它越好
declare -r user_agent='Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36'

function main() {

  # 这里把命令行第一个参数当作 URL
  declare url="$1"

  # 获取原始 HTML 源文件，把输出输出到 /dev/null，同时使用 gzip 进行解压，解压的内容放到
  # raw_html 变量中
  local raw_html=$(curl --user-agent "$user_agent" "$url" 2> /dev/null | gzip -d -)
local va=($(echo "$raw_html" | sed -nr $reg_data | jq "$reg_video_audio" | sed -nr 's/\"(.*)\"/\1/gp'))
  
  local tmpfile=$(mktemp) || exit 1
  curl --referer "$url" --user-agent "$user_agent" "${va[0]}" --output "$tmpfile-video.mp4"
  curl --referer "$url" --user-agent "$user_agent" "${va[1]}" --output "$tmpfile-audio.mp4"

  local title="$(echo "$raw_html" | sed -nr "$reg_title")"
  
  # 合并音、视频
  ffmpeg -i "$tmpfile-video.mp4" -i "$tmpfile-audio.mp4" -c copy "$title.mp4" -y -loglevel quiet
  
  # 删除脚本产生的文件爱你
  /bin/rm -r $tmpfile $tmpfile-audio.mp4 $tmpfile-video.mp4
}

main "$@"
```

至此脚本完成——非常简短。

## 测试

写完不代表一定成功，我们需要测试下载

```bash
$ ls
bl.sh
$ chmod u+x bl.sh # 赋予脚本可执行权限
$ ./bl.sh "https://www.bilibili.com/video/BV1Sg41137WR"
https://www.bilibili.com/video/BV1Sg41137WR
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 52.6M  100 52.6M    0     0  2945k      0  0:00:18  0:00:18 --:--:-- 2977k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5059k  100 5059k    0     0  2936k      0  0:00:01  0:00:01 --:--:-- 2938k
$ ls
 bl.sh  '【红楼梦】夜曲 宝玉.mp4'
```

完工！到这里， B站 视频下载就算完工了。

# 后语

简单来说，这次的教程不难，就是以下几个步骤：

1. 获取网页源码
2. 分析源码，获取音、视频链接
3. 下载音、视频
4. 合并音视频

知道了过程，使用其他编程语言也可以。使用 Shell 脚本看起来确实不是很优雅，为此我在这里附上用 Python 写的下载，不然简单的下载真的被我写的很难：

```python
#!/bin/python3
# -*- conding:utf-8 -*-

import os
import re
import sys
import json
import random
import requests

def main():

    print('Please Waiting...', flush=True)
    # 获取 URL
    url = sys.argv[1]
    # 设置 User-Agent
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36'
    }

    # 获取网页源码
    raw_html = requests.get(url, headers=headers)
    # 视频标题
    title = re.search('<h1 title=.*>(.*)</h1>', raw_html.text).group(1)
    # 获取目标数据，
    data = re.search('__playinfo__=(.*?)</script><script>', raw_html.text).group(1)
    # 把目标数据转化为 Python 可读的 JSON 格式
    json_data = json.loads(data)
    video_url = json_data['data']['dash']['video'][0]['baseUrl']
    audio_url = json_data['data']['dash']['audio'][0]['baseUrl']
    
    # 设置 Referer
    headers.update({'Referer': url})

    # 随即生成 1～100 作为基础文件名
    basefile_name = str(random.randint(1, 100))
    # 获取当前目录
    working_path = os.getcwd()
    # 创建临时文件名，
    video_file = os.path.join(working_path, f'{basefile_name}-video.mp4')
    audio_file = os.path.join(working_path, f'{basefile_name}-audio.mp4')

    # 下载视频和音频
    with open(video_file, 'ab') as video:
        for iter in requests.get(video_url, headers=headers).iter_content(1024):
            video.write(iter)

    with open(audio_file, 'ab') as audio:
        for iter in requests.get(audio_url, headers=headers).iter_content(1024):
            audio.write(iter)

    # 调用 FFmpeg 进行合并
    os.system(f'ffmpeg -i {video_file} -i {audio_file} "{title}.mp4" -c copy -y -loglevel quiet')

    # 删除创建的临时文件
    os.remove(video_file)
    os.remove(audio_file)

    print('Download END...')

if __name__ == '__main__':
    main()
```

Python 解析 JSON 非常慢，原本我写了一个进度条，结果发现解析 JSON 就很慢了，进度条基本等于没用，所以这里就没有加上。

即使是 Python 也需要使用 FFmpeg。如果使用的是 Windows 系统，需要设置 FFmpeg 的环境变量——或者把 `ffmpeg` 改为 `ffmpeg 的路径` 也行。例如，我使用 Windows 系统，并且下载的 FFmpeg 在 `C:\User\chunshuyumao\Downloads\ffmpeg.exe`， Python 脚本倒数第六行从

```python
os.system(f'ffmpeg -i......')
```

改为

```python
os.system(f'C:\\User\\chunshuyumao\\Downloads\\ffmpeg.exe -i.....')
```

如果觉得好玩——也许可以用用自己喜欢的语言实现一遍？
