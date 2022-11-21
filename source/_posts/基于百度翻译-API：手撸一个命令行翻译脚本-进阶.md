---
title: 基于百度翻译 API：手撸一个命令行翻译脚本--进阶
hide: false
tags:
  - 翻译
categories:
  - Linux
math: false
mermaid: false
date: 2022-03-26 09:00:31
index_img:
excerpt: Who care?
---

# 前言

正事从 2. 开始，前言只是发牢骚。

前面我们已经完成了翻译脚本的基本功能。所有的编程语言最大的问题一直是字符串，Shell 也一样。当我准备写下这一篇的时候还在想：要不要先出一篇介绍 Shell 编程的博客？Shell 语言和其他的编程语言也不太一样，我们更习惯叫它“脚本语言”，可惜这也不够准确。我觉得要真的做比较，可以看出做是高级的批处理语言。

前一阵子的翻译脚本，实际看来不复杂。原本打算一篇写完，结果还没写多少就突破三千字。不得不说，现在市面上各种叫嚣几分钟学会某种东西的声音太多，能有人抽出时间看完三千字不大可能，所以我直接断笔，把剩下的部分独立出来。这次尽量一篇写完。

# 参数

前一篇我们让脚本后面的参数直接充当 query 参数的值，也就是说我们只能这样使用我们的翻译脚本 `./mt.sh 翻译的文本`。可是看到 Linux 上的命令通常都有很多的命令参数，怎么看自己的脚本都太简单了。

![简单脚本](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203260911243.png)

此外，我们的脚本如果被其他人使用，别人不知道怎么使用怎么办？我们还希望自己的脚本可配置翻译语言和目的语言。鉴于这些要求，我们需要扩展之前的命令行参数。

这里复习一下：所谓命令行参数，指的是脚本之后跟着的字符。举个例子：`./mt.sh chun shu yu mao` 的命令行参数是 `./mt.sh`、`chun`、`shu`、`yu`和 `mao`，其中参数使用`$n`表示，n 是阿拉伯数字，不能超过 9 。

先确定我们需要什么参数。根据之前的分析，我们知道：from 表示源语言，to 表示目的语言，q 表示翻译文本。那好，我们就设置这些参数。我们习惯在参数前面加 `-` 表示参数，其中长参数使用两个短杆，短参数用一个。所以我们的目标是这样：

| 参数值 | 短参数 | 长参数  |
| :----: | :----: | :-----: |
|  from  |   -f   | --from  |
|   to   |   -t   |  --to   |
|   q    |   -q   | --query |

以后我们希望这样调用脚本：`./mt.sh -f en -t zh -q Yeah`，这样写的好处是可以随意调换参数位置，例如可以这样写：`./mt.sh -q Yeah -t zh -f en`

# 解析参数

要读取参数，我们需要一个循环。Shell 中的循环可以使用 while 。然后我们需要获取参数的个数，这样我们才能确定循环的次数。在 Shell 使用 `$#` 获取参数个数，所以我们的脚本框架是这样的：

```shell
while [ $# -gt 0 ]; do
   
    echo $1
    
    shift
done
```

在 Shell 中使用比较需要放到一个方括号`[]`里边，而且数值比较不能使用 `<` 等符号，而是英语简写。

|   中文   | 简写 |         英文          |
| :------: | :--: | :-------------------: |
|   小于   | -lt  |       less than       |
|   大于   | -gt  |     greater than      |
|   等于   | -eq  |         equal         |
| 小于等于 | -le  |  less than or equal   |
| 大于等于 | -ge  | greater than or equal |
|  不等于  | -ne  |       not equal       |

我们的 while 循环就是表示：当参数个数大于 0 个的时候开始是循环。要注意，因为 `$0` 默认表示脚本的名字，`$#` 不把它算上。也就说，`./mt.sh chun shu yu mao` 的参数总数是 4 而不是 5.

`shift` 表示偏移。这个有点难理解：还是拿 `./mt.sh chun shu yu mao` 来说，原本 `$1` 表示 `chun`，使用一个 `shift` 之后 `$1` 变成了 `shu`，再使用就是 `yu` 。`shift` 偏移之后参数个数就少了，所以我们处理完一个参数就用 `shift` 扔掉这个参数，直到所有参数都被处理完。

想要判断这个参数是 `-t` 还是 `-f` ，大家的想法可能是 `if` 语句。但是在这里， `if` 语句不好处理，推荐使用 `case` 语句。`case` 就像 C/C++ 语言中的 `switch`，Python3.10 之后的 `match` 。`case` 还可以用 `|` 表示或者，例如 `ab|bc` 表示匹配 `ab` 或者 `bc` ，语法是：

```shell
case 参数 in
	匹配1) 处理
	;;
	匹配2) 处理
	;;
	匹配3|匹配4) 处理
	;;
esac
```

​	既然如此，我们的脚本应该这样

```bash
while [ $# -gt 0 ]; do

  case $1 in
    '-t'|'--to') 
    to=$2
    shift
    ;;
    '-f'|'--from')
    from=$2
    shift
    ;;
    '-q'|'--query')
    query=$2
    shift
    ;;
  esac
  shift
done
```

我们判断了 `$1` 表示某个符号之后，直接读取它后边的参数，所以读取 `$2`，过程如下：

1. 执行脚本 `./mt.sh -f en -t zh -q text`
2. 进入 `case` ，判断 `$1` 是 `-f`，所以读取 `$2` 也就是 `en`。执行一次偏移，现在 `$1` 变成了 `en`.
3. 跳出 `case`，执行一次 `shift` ，现在 `$1` 是 `-t` .
4. 下一次循环，又回到第 2 步

在 Shell 中，我们有一个简写：如果前一步执行成功，我们可以用 `&&` 执行下一步，所以上面可以写成这样：

```shell
while [ $# -gt 0 ]; do

	case $1 in
		'-t'|'--to') to=$2 && shift ;;
		'-f'|'--from') from=$2 && shift ;;
		'-q'|'--query') query=$2 && shift ;;
	esac
	shift
done
```

现在我们的脚本如下，顺便测试一下：

```shell
#!/bin/bash
# encoding:utf-8

# 翻译 HTTPS 地址
declare -r basic_url="https://api.fanyi.baidu.com/api/trans/vip/translate"
# APPID
declare -r appid="xxxxxxxxxxxxxxxxxxxx"
# 密钥
declare -r token="xxxxxxxxxxxxxxxxxxxx"
# random code 
declare salt=$(openssl rand -base64 18|cut -c 1-10)

decalre from
declare to
declare query

while [ $# -gt 0 ]; do
  case $1 in
    '-t'|'--to') to=$2 && shift ;;
    '-f'|'--from') from=$2 && shift ;;
    '-q'|'--query') query=$2 && shift ;;
  esac
  shift
done

# 翻译
sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)

query=$(echo "$query" | tr -d '\n' | xxd -plain | sed 's/\(..\)/%\1/g')
query=$(echo $query | sed 's/ //g' )

result=$(curl "$basic_url?q=$query&from=$from&to=$to&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
result="${result##*dst\":\"}"
echo -e ${result%\"\}\]\}}
```

![中英互译](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261002830.png)

![中英互译](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261006717.png)

大多数时候，我们都是从英文翻译到中文，每次都写这么多的参数实在麻烦，所以我们在 `while` 之后写参数默认值：

```shell
while [ $# -gt 0 ]; do
  case $1 in
    '-t'|'--to') to=$2 && shift ;;
    '-f'|'--from') from=$2 && shift ;;
    '-q'|'--query') query=$2 && shift ;;
  esac
  shift
done

if [ -z $from ]; then 
  from='auto'
fi

if [ -z $to ]; then
  to='zh'
fi

sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)
```

`[ -z $from ]` 表示，如果 `$from` 是空( zero )的字符串，就执行 `if` 里边的语句。这样看起来有点不舒服，我们可以用上面学到的知识修改成下面的语句：

```shell
[ -z $from ] && from='auto'
[ -z $to ] && to='zh'
```

上面两种形式是等价的，大抵意思是说：源语言自动判断——百度翻译有这个功能，目标语言默认是中文。这样，如果我们可以简单使用 `./mt.sh -q 文本` 进行翻译。

![测试](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261018479.png)

![阿拉伯语](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261021075.png)

到这里，翻译部分基本讲完了。

# 拓展功能

大多数的命令行工具都可以使用 `命令 --help` 查看帮助，所以我们也假模假样搞一个帮助的文本。我的文本如下：

```shell
'-h'|'--help')
  echo "Usage: $0 [OPTION]" 
  echo "Valid options are:"
  echo -e " -q, --query\t\ttext want to translate"
  echo -e " -f, --from\t\tsource language. auto detect default"
  echo -e " -t, --to\t\tdestinate language, Chinese default"
  echo -e " -l, --list\t\tlist all support languages"
  echo -e " -i, --interact\t\tenter interact mode"
  echo -e " -c, --clear\t\tclear the screen"
  echo -e " cmd\t\t\tonly on interact mode. use to set settings, same as above."
  echo ""
  echo "example:"
  echo "$0 -f en -t zh -q 'I like it.'"
  echo "$0 'I like it'"
  echo ""
  echo "when use '$0 -i' you can type such command to change settings"
  echo "    >cmd -f zh -t en"
  ;;
```

把上面的文本放到 `while` 循环的  `case` 中。这里我还写了一些拓展，例如交互模式(interact mode)。

![帮助](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261028368.png)

![使用帮助](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261030188.png)

可以看到，列出帮助之后脚本仍旧执行翻译，所以后面出现报错。我们需要设置列出帮助之后直接退出脚本，在脚本中写下 `exit 0`。

```shell
  echo "    >cmd -f zh -t en"
  exit 0
  ;;
```

## 函数

`case` 分支 `--help`  看起来占位置，还不好看。我们可以写一个函数，把里边的语句单独提出来，然后调用脚本，这样脚本就可以写成：

```shell
function uasge(){
   # 这里写 echo ...
}

while [ $# -gt 0 ]; do
  case $1 in
    '-t'|'--to') to=$2 && shift ;;
    '-f'|'--from') from=$2 && shift ;;
    '-q'|'--query') query=$2 && shift ;;
    '-h'|'--help') usage && exit 0 ;;
  esac
  shift
done
```

Shell 中的脚本调用不用括号，直接列出函数名，如果有参数就直接放后面：`func para1 para2 para3`.

同理，完善 `--list` 列出支持的语言翻译的时候可以写成

```shell
function uasge(){
   # 这里写 echo ...
}

function language_support(){

  echo -e "name\t\tid"
  echo -e "Chinese\t\tzh"
  echo -e "  --Traditional\tcht"
  echo -e "  --Classical\twyw"
  echo -e "  --Cantonese\tyue"
  echo -e "English\t\ten"
  echo -e "Japanese\tjp"
  echo -e "Spanish\t\tspa"
  echo -e "Russian\t\tru"
  echo -e "Italic\t\tit"
  echo -e "Polish\t\tpl"
  echo -e "Danish\t\tdan"
  echo -e "Romanian\trom"
  echo -e "Hugarian\thu"
  echo -e "Korean\t\tkor"
  echo -e "Thai\t\tth"
  echo -e "Portuguese\tpt"
  echo -e "Greek\t\tel"
  echo -e "Bulgarian\tbul"
  echo -e "Finnish\t\tfin"
  echo -e "Slovene\t\tslo"
  echo -e "French\t\tfra"
  echo -e "Arabic\t\tara"
  echo -e "German\t\tde"
  echo -e "Dutch\t\tnl"
  echo -e "Estonia\t\test"
  echo -e "Czech\t\tcs"
  echo -e "Swedish\t\tswe"
  echo -e "Vietnamese\tvie"
  echo -e "and so on"
}

while [ $# -gt 0 ]; do
  case $1 in
    '-t'|'--to') to=$2 && shift ;;
    '-f'|'--from') from=$2 && shift ;;
    '-q'|'--query') query=$2 && shift ;;
    '-h'|'--help') usage && exit 0 ;;
    '-l'|'--list') language_support && exit 0 ;;
  esac
  shift
done
```

到这里，我们的脚本基本完善了，就差一个交互模式了。

## 交互模式

每次翻译我们都需要执行一次脚本太麻烦了，我们希望可以做到执行一次脚本后保留之前的设置继续翻译。此外，如果我们突然想改变翻译的一些设置，也可通过交互模式直接改变，而不用退出重新设置。这基本是交互模式的工作。

交互模式，顾名思义，我们就用 `-i` 和 `--interact` 代替。在你的 `case` 给交互模式赏一个分支：

```shell
  '-i'|'--interact')
  ;;
```

如果调用交互模式的指令是：`./mt.sh -i -f en -t zh`，我们希望脚本先分析完参数再执行交互模式。这样一来，`case` 的交互模式分支就不能用函数了。我们设置一个标志符，确认是不是交互模式。大概如下：

```shell

declare -i interaction=0
while [ $# -gt 0 ]; do
  # ...
    '-i'|'--interact') interaction=1 ;;
  # ...
done
```

后面我们再判断是不是交互模式。

```shell
[ -z $from ] && from='auto'
[ -z $to ] && to='zh'

[ $interaction -eq 1 ] && interact && exit 0
```

这次就需要写一个 interact 函数了。可以把函数放到 `while` 循环之前。

我们的想法是，交互模式一行一行读取我们的输入，就像这样：

![交互模式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261109882.png)

上面我设置了 `cmd` 作为交互模式修改设置的关键字，也就是而已使用 `cmd + 参数` 修改语言翻译的设置。

交互模式的函数如下：

```shell
function interact(){

  while read -p ">" query; do
    [[ -z "$query" ]] && continue
    [[ "$query" =~ "cmd " ]] && commands && continue
    
    # translate
  done
}
```

一行一行解释，`read` 是 Shell 读取输入的方式，就像 Python 的 input()、C++ 的 std::cin 、C 语言的 scanf() 等。`-p` 表示不换行，后面跟着提示符 `>` ，然后就是读取的字符。我们使用 `while` 读取循环，也就说，只有存在输入的时候才执行下面的语句，避免一直循环占用内存。

下一行表示，如果没有输入——例如我们直接回车——那继续循环，不执行命令。

再下一行表示，如果 `query` 包含 `cmd` 字符串，那就执行 commands 函数，然后继续循环，不执行后面的语句。

> Shell 中判断是否包含某一字符串使用 `=~` ，例如  A 字符串是否包含 B 字符串
>
> > [[ A =~ B ]]

再后面就是我们的翻译语句了。

**拷贝**我们的翻译语句，放到 `#translate` 的位置，就像这样：

```shell
function interact(){

  while read -p ">" query; do
    [[ -z "$query" ]] && continue
    [[ "$query" =~ "cmd " ]] && commands && continue
    
    sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)

    query=$(echo "$query" | tr -d '\n' | xxd -plain | sed 's/\(..\)/%\1/g')
    query=$(echo $query | sed 's/ //g' )

    result=$(curl "$basic_url?q=$query&from=$from&to=$to&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
    result="${result##*dst\":\"}"
    echo -e ${result%\"\}\]\}}
  done
}

declare -i interaction=0
while [ $# -gt 0 ]; do
  case $1 in
    '-t'|'--to') to=$2 && shift ;;
    '-f'|'--from') from=$2 && shift ;;
    '-q'|'--query') query=$2 && shift ;;
    '-h'|'--help') usage && exit 0 ;;
    '-l'|'--list') language_support && exit 0 ;;
    '-i'|'--interact') interaction=1 ;;
  esac
  shift
done

[ -z $from ] && from='auto'
[ -z $to ] && to='zh'

[ $interaction -eq 1 ] && interact && exit 0

# 翻译
sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)

query=$(echo "$query" | tr -d '\n' | xxd -plain | sed 's/\(..\)/%\1/g')
query=$(echo $query | sed 's/ //g' )

result=$(curl "$basic_url?q=$query&from=$from&to=$to&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
result="${result##*dst\":\"}"
echo -e ${result%\"\}\]\}}
```

可以看到，有好多重复的语句，特别是翻译的部分，所以我们写一个翻译的函数，直接调用：

```shell
function translate(){

  sign=$(echo -n "$appid$query$salt$token" | md5sum | cut -c 1-32)

  query=$(echo "$query" | tr -d '\n' | xxd -plain | sed 's/\(..\)/%\1/g')
  query=$(echo $query | sed 's/ //g' )

  result=$(curl "$basic_url?q=$query&from=$from&to=$to&appid=$appid&salt=$salt&sign=$sign" 2>/dev/null)
  result="${result##*dst\":\"}"
  echo -e ${result%\"\}\]\}}
}
```

然后脚本变成

```shell
function interact(){

  while read -p ">" query; do
    [[ -z "$query" ]] && continue
    [[ "$query" =~ "cmd " ]] && commands && continue
    
    translate
  done
}

declare -i interaction=0
while [ $# -gt 0 ]; do
  case $1 in
    '-t'|'--to') to=$2 && shift ;;
    '-f'|'--from') from=$2 && shift ;;
    '-q'|'--query') query=$2 && shift ;;
    '-h'|'--help') usage && exit 0 ;;
    '-l'|'--list') language_support && exit 0 ;;
    '-i'|'--interact') interaction=1 ;;
  esac
  shift
done

[ -z $from ] && from='auto'
[ -z $to ] && to='zh'

[ $interaction -eq 1 ] && interact && exit 0

# 翻译
translate
```

![交互](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/202203261129188.png)

接下来写 `commands`. 编写 `commands` 函数之前我们知道，其内部的解析实际上和解析脚本参数基本一样，所以我们要做的就是去除 `cmd ` 字符串，然后把后面的参数发给原本的参数解析函数——原本只是普通语句，我们也需要把它封装起来，所以就变成下面这样：

```shell
# ...

function commands(){

  local args=$(echo "$query" | sed 's/cmd\ //g')
  query=''
  settings $args
}

function interact(){
	# ...
}

declare -i interaction=0

function settings(){

  while [ $# -gt 0 ]; do
    case $1 in
      '-t'|'--to') to=$2 && shift ;;
      '-f'|'--from') from=$2 && shift ;;
      '-q'|'--query') query=$2 && shift ;;
      '-h'|'--help') usage && return 0 ;;
      '-l'|'--list') language_support && return 0 ;;
      '-i'|'--interact') interaction=1 ;;
      *） query=$1 ;;
    esac
    shift
  done
}

settings $@

[ -z $from ] && from='auto'
[ -z $to ] && to='zh'

# ...
```

鉴于放到了函数里，我们就不使用 `exit 0` 了，而是改成 `return 0` . 而且，我们的 `while`循环多了一个分支 `*` ，这个表示，如果没有任何标志的话，默认直接翻译无法解析的参数。其实就是一句话：脚本之前必须用 `-q` 指明翻译的文本，现在不用了，可以这样用 `./mt.sh text`。

至此，我们的脚本写完了。当然，眼尖的同志看到，我还没有实现 `clear` 指令呢！没事，这让你们自己探索。

# 后语 

又超过三千字了，这使我不得不结束这篇文章。脚本可拓展的其实还很多，有兴趣可以试试。我附上两个脚本——mt1.sh 和 mt.sh. 前者是写这篇文章使用的脚本，后者是我之前写的完整的脚本。因为蓝奏云不支持 `sh` 格式，所以我重命名为 `mt1.sh.txt`  和 `mt.sh.txt`。下载之后只需要把扩展名中的 `.txt` 删掉就可以了。

# 相关脚本

脚本: https://wwb.lanzoub.com/b017dhqaf 
密码: chunshuyumao
