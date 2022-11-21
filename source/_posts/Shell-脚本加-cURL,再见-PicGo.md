---
title: Shell 脚本加 cURL, 再见 PicGo
hide: false
tags:
  - Shell
categories:
  - Manjaro
math: false
mermaid: false
date: 2022-06-27 18:30:23
index_img:
excerpt: 抛弃 PicGO，自己动手丰衣足食！
---

# 前言

谈谈最近的小事。

## Sway-平铺窗口管理

GNOME 桌面 的极简主义让我很满意。为了让 GNOME 更符合自己的审美，我添加了七八个插件。正因此，最近多次重启电脑，因为这货居然会卡住没反应！考虑到GNOME 一直以内存泄漏出名，我打算试试平铺窗口管理（Tiling Window Manager）。

所谓"窗口平铺"，就是系统自动将打开的各种窗口排好，而不是一般系统的叠加（正确来说叫做"浮动（Floating）"）。

Linux 上平铺窗口管理最著名的就是 [i3wm](https://i3wm.org/ "i3wm")，我选择 [Sway](https://swaywm.org/ "Sway")，适用于 [Wayland](https://wayland.freedesktop.org/ "Wayland") 的 i3。[Manjaro Linux](https://manjaro.org/ "Manjaro Linux") 社区有在维护 Sway 版，可惜我最开始的时候并没有选择 Sway。

![官网截的图，因为我的 Sway 已经被我卸了。](http://101.200.84.36/images/2022/06/27/202206271741491.png "官网截的图，因为我的 Sway 已经被我卸了。")

平铺有利于桌面空间的利用，全快捷键操作也可以让人脱离键盘，看起来极富极客（Geek）风格，也就是装～。其实 Windows 系统也有平铺的功能，以前都是快捷键，Windows11 之后直接在最大化按钮加上平铺。

安装 Sway 主要借助的是 [Arch Linux Wiki](https://wiki.archlinux.org/ "Arch Linux Wiki")。没办法，这家伙的文档贼丰富。Sway 的配置太难受了，包括 i3，由于不希望安装太多第三方软件，我真的是从头到脚写了各种脚本，真的要吐。折腾到后面，感觉虽然有成就感，但是自己不愿折腾了，回到了 GNOME。继续使用我的 GNOME 拓展 [gTile](https://extensions.gnome.org/extension/28/gtile/ "gTile")。

## GitHub-Copilot

[GitHub Copilot](https://github.blog/2022-06-21-github-copilot-is-generally-available-to-all-developers/ "GitHub Copilot 消息来源") 是一个可以根据注释生成代码的人工智能。Copilot，顾名思义，当你的副手。GitHub 对外开放不代表免费，实际上价格极其感人。好消息是，对于开源项目维护者和学生、老师来说，这东东是免费的。当然，前提是你得完成学生认证。GitHub 的[学生认证](https://education.github.com/ "学生认证")其实好处多多（认证的邮箱建议使用学校邮箱），可以拿到各种优惠包。由于自己用不上，所以还没有进行认证。白嫖只是白白占有资源，何必呢。

GitHub Copilot 支持 Visual Studio、Visual Studio Code 和 NeoVIM。如果感兴趣，也可以到[官网](https://github.com/features/copilot/ "GitHub Copilot")瞄瞄这家伙如何工作。当然，如果进行学生认证的话还可以免费使用。

![GitHub Copilot](http://101.200.84.36/images/2022/06/27/202206271808381.png "GitHub Copilot")

## PicGo

[PicGo](https://www.picgo.net/ "PicGO") 才是今天的主角，准确说是即将退场的主角。PicGo 是一款集成各类图床的上传工具，可以通过 图床API（Application Programming Interface，应用编程接口）轻松实现图片的上传。具体就不介绍了。

是这样的，Linux 系统 PicGo 的 AppImage 包高达 100M。不得不说，对于有洁癖的人来说，是可忍孰不可忍？自己使用的图床主要包括 [Chevereto](https://chevereto-free.github.io/ "Chevereto")、[GitHub](https://github.com/ "GitHub") 和 [SM.MS](https://sm.ms/ "SM.MS")，主要是前两者，后者只是以前用的。自己用不上 PicGo 的多功能，而且 PicGo 还那么臃肿，由此萌发了使用脚本自己写一个上传工具的念头。

说到脚本，很多人想到的自然是 Python，不过很可惜，Python 在我眼中就是臃肿的另一个代表，虽然自己的系统就有Python。依赖越少的工具，越深得我的心，所以像 Python 这类脚本语言基本可以散了，实际上 Lua 可以考虑。Linux 系统离不开 Shell，所以后面决定用 Shell/Bash。

# 替代 PicGo

写一个替代 PicGo 的脚本其实还是很简单的。Windows 系统没必要看，因为 Windows 系统的生态确实太好了，没必要折腾。

## 准备

简单介绍用到的东东：

1. Linux GNOME 桌面环境。指定 GNOME 系统是为了使用 GNOME 的全局自定义快捷键。因为我需要减少依赖，不可能为了设置全局快捷键下载一个软件或者程序。
2. Xclip，Linux X桌面协议系统（Xorg）的剪贴板程序。Linux 不是 X协议 就是 Wayland协议。由于 Wayland协议 现在生态不咋地，为了兼容 Xorg 生出了 XWayland 这个缝合怪（对的，就是两者的结合），因此直接使用Xclip 就行，没必要用 wl-clipboard，徒增兼容问题。这是 Linux 标配。
3. Bash/Shell，最普遍的 Shell 环境，Linux 标配。
4. SED，Stream Editor，流式编辑器，Linux 标配。
5. [cURL](https://curl.se/ "cURL")，非常强大的命令行文件传输工具，Linux 标配。
6. notify-send，命令行或者脚本下的通知工具，非常简单。不清楚是不是有桌面的 Linux 都会带着，但是一般确实如此。
7. JQ，命令行环境处理 JSON 格式文件的利器，见鬼，以前自己不愿安装，纯手工解析 JSON,后面学乖了，好好使用 JQ 吧。一般需要安装，一般使用命令行的都会安装。
8. GitHub 帐号，可选，用于保存图片，也就是做图床。
9. SM.MS 帐号，可选，做图床。

上面的工具基本都是一个正常的 Linux 系统自带的，除了 JQ 可能没有。还有，那两个帐号可不自带。

## 搭建框架

找一个放脚本的目录，例如我喜欢把自己的脚本放到 `$HOME/.scripts` 目录下，创建一个 Shell 脚本。

```bash
$ cd ~/.scripts # 进入脚本目录
$ pwd # 输出当前目录
/home/chunshuyumao/.scripts
$ vim upload.sh

# 以下为脚本内容
#!/bin/bash
set -euo pipefail

function main() {

}

main "$@"
```

创建脚本之后先写一个主函数(main)，让我们的脚本看起来更加清晰。这些内容之前的博客都出现过了，这就不解释了。

我们使用的是 GNOME 桌面系统自带的截图工具。GNOME42 之后，这个工具的快捷键是 `prt`，也就是键盘上的打印键。默认截图之后会保留在剪贴板，接下来我们需要从剪贴板获得图片。

不过想想，获取的图片保存到哪里？由于我们把图片上传到图床，本地保留一份看起来没必要，所以建议创建一个临时文件，上传之后删掉。我就是这么做的。实际上 PicGo 也是这样做的（之前，现在有内置方法）。从剪贴板获取图片使用以下命令：

```bash
xclip -sel clip -t image/png -o > image.png
```

当然，你要是乐意，可以写全参数：

```bash
xclip -selection clipboard -target image/png -out > image.png
```

懒人就不会选择长命令。长命令形式一目了然，我们从（selection）剪贴板（clipboard）获取目标图片PNG格式（target image/png），输出（out \>）到 `image.png`。好家伙，完全不用解释哦！

这样在脚本所在目录就会出现一个 `image.png` 文件。

![从剪贴板获取图片](http://101.200.84.36/images/2022/06/27/202206272005261.png "从剪贴板获取图片")

可是这样不优雅，图片直接在脚本所在的地方生成非常恶心。好在我们可以做得更好！一个优雅的方式是，创建一个临时目录，把所有的临时文件放到这个目录中，等到任务完成之后再删去目录。好巧不巧，刚刚好有一个这样的命令符合我们的预期，它就是 `mktemp`。通过查阅使用手册，我们可以知道，`mktemp` 可以这样创建临时文件夹：

```bash
$ mktemp -d -t temp.XXXXXXXXXX
/tmp/tmp.e7Y2FzNj5Q
```

`-d` 表示生成临时文件夹，不加的话会生成临时文件。`-t` 表示在临时文件夹（/tmp）下生成。 `temp.XXXXXXXXXX` 是生成的文件夹名，其中 `temp.` 是随便取的，这是我自己选的；`XXXXXXXXXX` 代表生成多少个随机的字符，你要乐意，写一万个也行。`mktemp` 生成临时文件夹之后会返回文件夹的路径。

为了让自己的脚本更加健壮，我们需要确认 `mktemp` 确实成功创建了文件夹，否则（||）直接退出（exit）。除此之外通过 `date` 创建图片的名字，然后和临时文件夹整合到一起作为图片的路径。因此我们的脚本现在变成了这样：

```bash
#!/bin/bash
set -euo pipefail

# 按时间给图片命名，依次是 年 月 日 时 分 秒 星期
# [Y]ear [m]onth [d]ay [H]our [M]inute [S]econd  u --> 鬼知道这是什么的缩写
declare -r image_name="$(date +%Y%m%d%H%M%S%u).png"
# 如果创建临时文件夹失败，就使用 exit 1 退出
declare -r tmpdir="$(mktemp -d -t temp.XXXXXXXXXX)" || exit 1
declare -r path2image="$tmpdir/$image_name"
# 通知持续时间为 3 秒
declare -r expire=3000

function main() {

  # Notifies when there is not image in clipboard, and exits.
  xclip -sel clip -t image/png -o > "$path2image" || {
    notify-send -t $expire -i 'dialog-warning' 'WARNING' "There is not image exiting in clipboard."
    exit 1
  }
}

main "$@"
```

`||`表示前面的命令失败的话就执行后面的命令。其实这些在之前的文章都有讲到。`notify-send` 用于弹出通知，大概这样使用：

```bash
#             通知几秒后关闭？          图标是啥？              标题      内容
$ notify-send --expire-time milliseconds --icon 'dialog-icon' 'Caption' 'Details'
```

脚本中的意思是：从（selection）剪贴板（clipboard）获取（target）PNG格式的图片（image/png），并输出（out >）到 `path2image` 中。如果获取失败（||），以警告图标（icon dialog-warning）发送标题为"WARNING"，内容为"巴拉巴拉"的通知（notify-send），通知时间持续 3 秒。

上传到 SM.MS 还是 GitHub 是不同的选择，所以待会写两个函数，一个传到 SM.MS，一个传到 GitHub。我们希望调用函数之后，函数返回上传成功后图片的地址，也就是：

```bash
local image_url="$(upload2github)"
```

获得的地址以 Markdown 格式传到剪贴板，这样就可以直接粘贴了，也就是

```bash
echo "![](${image_url})" | xclip -sel clip
```

最后再发送(notify-send)一个通知(dialog-information)，说明（details）成功（caption）了：

```bash
notify-send -t $expire -i 'dialog-information' 'SUCCESS' '${image_url}'
```

处理完之后，别忘了删除我们的临时文件夹，写一个 clean 函数，在 main 函数里面显式调用，或者当捕捉(trap)到退出(EXIT)信号时被动调用。现在来看看我们的脚本：

```bash
#!/bin/bash
set -euo pipefail

trap clean EXIT

# 按时间给图片命名，依次是 年 月 日 时 分 秒 星期
# [Y]ear [m]onth [d]ay [H]our [M]inute [S]econd  u --> 鬼知道这是什么的缩写
declare -r image_name="$(date +%Y%m%d%H%M%S%u).png"
declare -r tmpdir="$(mktemp -d -t temp.XXXXXXXXXX)" || exit 1
declare -r path2image="$tmpdir/$image_name"
# 通知持续时间为 3 秒
declare -r expire=3000

function clean() {
  /bin/rm -rf $tempdir
}

function main() {

  # Notifies when there is not image in clipboard, and exits.
  xclip -sel clip -t image/png -o > "$path2image" || {
    notify-send -t $expire -i 'dialog-warning' 'WARNING' "There is not image exiting in clipboard."
    exit 1
  }
  
  # 默认上传到 GitHub
  local image_url="$(upload2github)"
  # 复制到剪贴板
  echo "![](${image_url})" | xclip -sel clip
  notify-send -t $expire -i 'dialog-information' 'SUCCESS' "${image_url}"
  
  clean
}

main "$@"
```

框架搭建完毕！后面如果有其他的图床，多写一个函数就完了。此外， PicGo 干的其实也不多就是这件事，只不过是人家是跨平台可视化的，而我写的这个仅限于 类Unix 系统。

## 编写 upload2github 函数

具体实现上传函数。为了普适性，我拿 GitHub 图床做一个例子。首先要准备几样东西

1. 图床仓库
2. 私人令牌（Token）

这里就不介绍如何申请 GitHub 仓库了，默认你建立了自己的仓库。这个仓库名一定要准确。例如我的仓库 `202203`：

![202203](http://101.200.84.36/images/2022/06/27/202206272146001.png "202203")

有了这个我们可以写函数了：

```bash
function upload2github() {

  #################设置####################
  local repo='202203'
  local username='chunshuyumao'
  local token='xxxxxxxxxxxxxxxxxxxxxx'
  #################可选####################
  local cdn="https://cdn.jsdelivr.net/gh/$username/$repo@master"
  #################设置####################
}
```

私人令牌（token）需要到帐号设置里获取。CDN（Content Delivery Network，内容分发网络），用于加速 GitHub 图床的图片显示，毕竟国外的东西，访问难免慢，使用 CDN 加速就会快一点。除了设置网起来的配置，其他都不要动。

参考 [GitHub官方API使用说明](https://docs.github.com/en/rest/repos/contents "GitHub API PUT") 上传图片。

![GitHub API](http://101.200.84.36/images/2022/06/27/202206272200431.png "GitHub API")

语法如下：

```bash
curl \
  -X 'PUT' "$github_url" \
  -H 'Accept: application/vnd.github.v3+json' \
  -H "Authorization: token $token" \
  -d '{ "message": "upload", "content": "'$content'" }'
```

`-X` 表示请求的方式，这里使用 `PUT`。`-H` 用于构建请求的头部，也就是 `header`。`-d` 是上传的数据，即网络请求的 `data`。

`$github_url` 是一个 API URL，如下：

```Bash
local github_url="https://api.github.com/repos/$username/$repo/contents/$image_name"
```

`token` 我们已经见过了。`$content` 呢？官网说的很清楚，是一个通过
`base64` 编码的字符串。这样获取：

```Bash
local content="$(base64 -w 0 $path2image)"
```

`-w 0` 表示不用换行（no wrap）。习惯上，文本文件一行限制 70 个字符，这称为 `wrap`。无缘无故添加换行符，后面难以处理，所以我们设置不换行。

请求之后返回的结果是 JSON 格式，保存到一个变量，然后用 JQ 进行处理：

![](http://101.200.84.36/images/2022/06/27/202206272209181.png)

```bash
local message="$(curl \
  -X 'PUT' "$github_url" \
  -H 'Accept: application/vnd.github.v3+json' \
  -H "Authorization: token $token" \
  -d '{ "message": "upload", "content": "'$content'" }' \
  2> /dev/null | jq 'commit..message?' sed -r 's/\"(.*)\"/\1/g')"

if [[ "$message" != "upload" ]]; then
  notify-send -t $expire -i 'dialog-error' 'ERROR' "$message"
fi
exit 1
```

我们不需要使用返回的结果，只需要判断是否成功。文档说明，如果上传成功，返回的 message 是我们之前的 message 的内容，我们只需要判断两个 message 想不相同即可。如果不同，直接报错通知。

`2> /dev/null` 表示把错误输出到 `/dev/null` 文件。`|` 叫做"管道符"，表示把左边的输出当作右边的输入。`jq '.message?'` 的意思是获取 JSON 格式第一层的 `message` 信息，问号表示如果没有这个信息，那就返回 `null` 代替，相当于:

```bash
if message exists in JSON, then return message else return null
```

上传成功之后，我们直接返回图片的链接：

```bash
echo -n "$cdn/$image_name"
```

Shell 脚本的返回和其他语言的返回不大一样，其实这个是打印，而不是返回。最后我们的脚本是：

```bash
# 函数描述:
#   上传图片到 GitHub 图床
# 参数: 无
# 返回值: 图片的链接，或者弹出失败后的信息
function upload2github() {

  ####################设置#######################
  local repo='202203'  # 这是你的仓库名
  local username='chunshuyumao' # 这是你的帐号名，现在写的是我的
  local token='xxxxxxxxxxxxxxxxxxxxxx' # 你的私人令牌，token
  ################可选 CDN 加速###################
  local cdn="https://cdn.jsdelivr.net/gh/$username/$repo@master"
  ####################设置########################

  local content="$(base64 -w 0 $path2image)"
  local github_url="https://api.github.com/repos/$username/$repo/contents/$image_name"
  local message="$(curl \
    -X 'PUT' "$github_url" \
    -H 'Accept: application/vnd.github.v3+json' \
    -H "Authorization: token $token" \
    -d '{ "message": "upload", "content": "'$content'" }' 2> /dev/null | jq '.message?')"

  if [[ "$message" != "upload" ]]; then
    notify-send -t $expire -i 'dialog-error' 'ERROR' "$message"
    exit 1
  fi

  echo -n "$cdn/$image_name"
}
```

GitHub 函数写完。后面来看看 SM.MS。

## SM.MS 函数

SM.MS 图床也需要准备一个 `token`，主要结构如下：

```bash
# 函数描述:
#   上传图片到 SM.MS 图床
# 参数: 无
# 返回值: 图片的链接，或者返回失败后的信息
function upload2smms() {

  #################SETTINGS START#################
  local smms_url="https://sm.ms/api/v2/upload"
  local token="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  #################SETTINGS END###################

  local result="$(curl \
    -X 'POST' "$smms_url" \
    -H 'Content-Type: multipart/form-data' \
    -H "Authorization: Basic $token" \
    -F "smfile=@${path2image};type=image/png;filename=${image_name}" 2> /dev/null)"

  local image_url="$(echo -n "$result" | jq '.data.url?' | sed -r 's/\"(.*)\"/\1/g')"
  if [[ "$image_url" == "null" ]]; then
    notify-send -t $expire -i 'dialog-error' "ERROR" "$(echo "$result" | jq '.message?')"
    exit 1
  fi

  echo -n "$image_url"
}
```

这个也很简单，参考[官网](https://doc.sm.ms/ "SM.MS API")。`-F` 参数是 `cURL` 传递 `form-data` 的方式，因为官网说了上传的是文件。一般上传文件直接使用 `-F`。

[SM.MS上传](http://101.200.84.36/images/2022/06/27/202206272237441.png "SM.MS上传")

`jq '.data.url?'` 表示从 JSON 中获取 data 下的 url 内容。大概是这样的布局：

```bash
{
    "data": {
        "usr": "blabla"
    }
}
```

问号仍然表示没有这东东的话返回 `null`，后面的 `message` 也一样。

`sed -r 's/\"(.*)\"/\1/g'` 用于删掉链接两边的双引号（""），因为链接两边不应该有双引号。

最后判断获取的是不是链接，不是的话就通知错误，并退出。轻轻松松，脚本完成！

## 设置全局快捷键

这里只说 GNOME 的全局快捷键。打开`gnome-center-control`，或者直接找设置（settings）

![设置](http://101.200.84.36/images/2022/06/27/202206272247001.png "设置")

选择`键盘（Keyboard）=> 键盘快捷键（Keyboard Shortcuts）=> 浏览与自定义快捷键（View and Custumize Shortcuts）=> 自定义快捷键（Custom Shortcuts）`，点击`➕`：

![添加](http://101.200.84.36/images/2022/06/27/202206272250411.png "添加")

![添加快捷键](http://101.200.84.36/images/2022/06/27/202206272251341.png "添加快捷键")

`描述（Name）` 只是描述，随便写啥；`命令（comamnd）`写你的脚本所在的路径，最后点击 `快捷键（Shortcut）`进行快捷键设置。Over！

# 后语

最后，把 PicGo 给卸了，毕竟，自己写的脚本才几K？开玩笑，其实 PicGo 除了简单，还有可视化、预览、删除图床图片等等的功能，自己的这个脚本只是阉割版罢了。不得不说，Linux 下真的可以自己动手丰衣足食。
