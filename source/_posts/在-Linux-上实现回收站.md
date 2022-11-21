---
title: 在 Linux 上实现回收站
hide: false
tags:
  - 脚本
  - 技巧
categories:
  - Linux
math: false
mermaid: false
date: 2022-04-29 15:41:55
index_img:
excerpt: Linux rm 命令的救星
---

# 前言

在 Windows 系统的时候，我会习惯使用快捷键 <kbd>Shift</kbd> + <kbd>delete</kbd> 进行直接删除，感觉这很酷 —— 不用经过回收站。当然，时不时还是会后悔自己错删了某个文件，但是 Windows 上有太多的数据恢复软件，完全不用担心。使用 Linux 之后，我很习惯命令行上的 `rm` 命令，直接把数据删个干净。有次错把自己的一个重要文件给删了，差点没救回来。那时候起，我开始考虑实现一个脚本，就像 Windows 那样 —— 不直接删除数据，而是挪到一个地方。

# 想法

Linux 大部分桌面系统是有回收站的 —— 至少我使用的 GNOME 桌面系统是这样的。回收站的位置一般是 `~/.local/share/Trash` 。其中  `~` 表示家目录，例如我的家目录是 `/home/chunshuyumao` 。通过文件管理器删除一个文件，我们可以在回收站目录下看到两个文件: `info` 和 `files` 。前者是被删文件的信息，例如路径和删除时间；后者是被删文件移动到的位置。


![空回收站](http://101.200.84.36/images/2022/04/29/202204291614948.png)


图形界面删除一个文件之后，可以在回收目录下找到删除的文件和信息：

```bash
$ tree
.
├── expunged
├── files
│   └── test.sh
└── info
    └── test.sh.trashinfo

3 directories, 2 files
```

查看 `trashinfo` 文件不难发现，这其实是被删文件的信息:

```bash
$ cat info/test.sh.trashinfo
[Trash Info]
Path=/home/chunshuyumao/test.sh
DeletionDate=2022-04-29T16:15:24
```

现在，我们要做的，就是写一个脚本，做到和图形界面一样的效果：`rm` 不是直接删除，而是移动到固定位置，同时生成一个被删文件的信息。

# 实现

想法很简单，直接使用 `mv` 代替 `rm` 。不过我们还需要实现 `trashinfo` 文件，以便可以模仿图形界面恢复文件。

## 搭建框架

首先创建一个文件，命名为 `mv2trash.sh` ，意思是移动到回收站而不是直接删除。至于脚本放哪里，我的建议是在自己的家目录创建一个 `.scripts` 文件夹，然后在里边放自己写的脚本或配置。

```bash
$ mkdir ~/.scripts
$ vim mv2trash.sh
```

习惯性地在脚本第一行添加 `#!/bin/bash` ，之前说过这是例行公事。

首先，我们需要创建一些常量( read only ，符号是 `-r` )：

```bash
#!/bin/bash

# TrashPath 表示被删文件的路径
declare -r TrashPath=$HOME/.local/share/Trash/files
# InfoPath 表示信息文件的位置
declare -r InfoPath=$HOME/.local/share/Trash/info

# 确认回收站的位置是否存在
[ ! -e "$TrashPath" ] && mkdir -p "$TrashPath"
[ ! -e "$InfoPath" ] && mkdir -p "$InfoPath"

function main() {

  while [ $# -gt 0 ]; do
    # 做一些操作
    # command
    shift
  done
}

# 调用函数， "$@" 是传递所有参数
main "$@"
```
`$HOME` 表示家目录，一般是 `/home/username` ，比如我的家目录 `/home/chunshuyumao` ，使用 `$HOME` 而不是绝对路径是为了通用性 —— 当你移动这个脚本到其他电脑的时候它仍然指的是家目录，即使你的用户名并不是 `chunshuyumao`。

`[ -e TrashPath"  ]` 判断路径是否存在，`-e` 是 `exist` 的缩写，前面添加 `!` 则表示否定。`mkdir` 是 <ruby>创建目录<rt>make directory</rt></ruby>的缩写；`-p` 是 `--parents` 的短命令形式，如果父目录不存在就创建而不是报错。`&&` 表示前一个命令为真就执行后一个命令。因此，只有路径不存在的时候才会创建路径。

> -p 可能说的比较奇怪。其实是这样的，mkdir 命令只能创建一级目录，加上 -p 就可以创建多级目录。
> mkdir file && mkdir file/filea && mkdir file/filea/fileb 这就是之能够创建一级目录怎么创建 file/filea/fileb
> 如果使用 -p 或者 --parent 的话，可以直接通过 mkdir -p file/filea/fileb 一气呵成。
> 
> . 开头的文件是隐藏文件，使用 ls 是看不见的，需要使用 ls -a 才可以。 

后面我们创建了一个函数 `main` 并调用了它。同时记得，我们把脚本的参数也一并传递了过去。在 `main` 函数中，我们判断，只要参数不为零，就一直循环，同时用 `shift` 移除使用过的参数。

## 实现细节

首先看看我们的实现：

```bash
function main() {

  while [ $# -gt 0 ]; do
  
    local filename="$(basename $1)"
    local filepath="$(cd $(dirname $1); pwd)/$filename"
    local trashinfofile="$InfoPath/$filename.trashinfo"
    local deletiondate="$(date +%Y-%m-%dT%H:%M:%S)"
    
    # 使用系统的移动命令移动文件到指定地点
    /usr/bin/mv "$filepath" "$TrashPath"
    
    # 创建一个 trashinfo 文件
    # 并写入相关信息
    cat >> "$trashinfofile" << _EOF_
[Trash Info]
Path=$filepath
DeletionDate=$deletiondate
_EOF_
    echo "rm: remove $filepath to $TrashPath "
    shift
  done
}
```

上面我们创建了四个局部变量 —— 也就是 `local` 修饰的那几个变量。这里有几个函数介绍一下：

1. `basename` ，语法：`basename path_to_file` 。这个函数返回给予路径的文件名。我们不知道使用者提供的是什么路径，可能是绝对路径 `/home/chunshuyumao/test.txt` ，也可能是相对路径 `~/test.txt` ，但使用这个 `basename` 函数，它会正确返回 `test.txt` 这个文件名。
2. `dirname`，语法：`dirname path_to_file`。该函数会返回给予路径的路径名。如果我们提供的是 `/home/chunshuyumao/scripts/test.sh`，函数会返回 `/home/chunshuyumao/scripts`；如果提供的是当前目录下的 `test.sh` ，函数会返回 `.` —— 一个小圆点表示当前路径。为了得到当前位置的绝对路径，我们需要进入这个路径，然后使用 `pwd` 打印当前路径 —— `pwd` 是 <ruby>打印工作目录<rt>Print Working Directory</rt></ruby>的缩写。如果不理解，只需要知道这种方法可以获取文件绝对路径就行了。最后通过拼接，形成了文件的绝对路径。
3. `date`，语法：`date formating_string`。这个函数可以获取当前时间 ，并按照 `formating_string` 进行格式化。在上面的脚本中，我们希望输出“年-月-日T时:分:秒”。

获取变量之后使用系统的移动命令移动文件或者文件夹到指定位置即可。注意，这里的移动命令 `mv` 没有简单使用 `mv` 而是 `/usr/bin/mv` ，这是防止有其他的别名影响。

最后的 `cat` 一行可能比较难以理解。这里引入一个新的语法糖，我会慢慢解释。
如果我们需要创建一个新的文件，并在其中输入一些文字，最常用的方法是打开编辑器然后输入文字、保存、退出。有些时候我们输入的内容太少或者不想大费周章打开一个编辑器，这种时候重复上面的操作就难免有点复杂了。好在 Linux Shell 上重定向符号 `>`、`<`等可以满足我们的需求。下面介绍几种重定向符号：

1. `>`输出重定向符。语法：`command > output_file`。左边是 <ruby>命令<rt>Command</rt></ruby> ，右边是 <ruby>保存文件<rt>Output File</rt></ruby> 。重定向输出，如果指定的文件不存在会自动生成；可是也要注意：如果存在目标文件，命令会清空文件中的所有内容，然后再把结果写入到目标文件。一句话：如果之前就有这个保存文件，那么保存文件内的原始内容将被删掉。 `ls` 列出当前目录下的文件，使用重定向符号试试：

```bash
$ ls > list.txt
$ ls
Desktop    Downloads  Music     Public     Videos
Documents  list.txt   Pictures  Templates  Zotero
$ cat list.txt
Desktop
Documents
Downloads
list.txt
Music
Pictures
Public
Templates
Videos
Zotero
```


![重定向之后 ls 输出的内容到了 list.txt 当中](http://101.200.84.36/images/2022/05/08/202205081622531.png "重定向之后 ls 输出的内容到了 list.txt 当中")

可以看到，使用 `>` 重定向符号之后 `ls` 命令不再把结果输出到屏幕上，而是输出到我们指定的文件 `list.txt` 中。这个时候如果不存在指定的文件，命令就会重建一个文件。这个功能其实很有用，特别是当我们希望把命令的操作结果保存时。

上面说了，如果这个文件本来就有内容了，这个操作会抹除原始的内容，所以专门有个 `>>` 符号（追加重定向）作为辅助 —— 这个符号只是在文件末尾追加内容，而不是抹除然后再输入：

![使用输出重定向会直接抹除原来的内容](http://101.200.84.36/images/2022/05/08/202205081640928.png)

![使用追加重定向后 list.txt 的内容增加而不抹除原内容](http://101.200.84.36/images/2022/05/08/202205081642542.png)

2. `<` 输入重定向符。语法：`command < input_file`。读取右边的文件，并把结果传给左边的命令。下面借助 `wc -l` 计算 `list.txt` 的行数：

![输入重定向](http://101.200.84.36/images/2022/05/08/202205081636770.png "输入重定向")

同理，也有一个 `<<` 不过这个一般不单独出现，而是出现在一个被称为 “Bash Here Document” 的东西。这个东西的语法是：

```bash
command << InputComesFromHere
...
...
...
InputComesFromHere
```

上面的 `.` 表示一些文字。其中 `InputComesFromHere` 是一个符号，表示从这里开始到下一个同样的符号结束的都是想要输入重定向的文字。这个 `inputComesFromHere` 符号一般随便选择 —— 通常我们会选择 `EOF` 或者 `_EOF_`，表示 <ruby>文件结束符<rt>End Of File</rt></ruby>，因为一般的文件末尾都使用它作为标识。到这里，我上面的命令就可以理解了：

```bash
    cat >> "$trashinfofile" << _EOF_
[Trash Info]
Path=$filepath
DeletionDate=$deletiondate
_EOF_
```
把两个 `_EOF_` 之间的内容追加到文件 `"$trashinfofile"` 中，这就很直观了。

我们的脚本基本已经完成，输入 `chmod u+x mv2trash.sh` 赋予脚本可执行权限，然后尝试删除：

![使用我们的脚本](http://101.200.84.36/images/2022/05/08/202205081709669.png)

现在 `.local/share/Trash` 下面有两个文件夹，其中分别装着刚刚删除的文件 `list.txt` 和 `list.txt.trashinfo`。以后我们删除的文件和生成的文件信息文件也会挪到这些位置。到这里，我们的框架基本搭完了，其实还可以补充更多的细节，例如交互模式 —— 用于确认删除。

## 实现恢复

删除文件之后，我们希望可以做到一键恢复，这就不得不再写一个脚本 —— 如果不愿意再写一个脚本，可以选择直接移动文件到原来的位置，然后删除 `Trash Info` 文件。不过我这里还是实现一下自动恢复：

![恢复效果](http://101.200.84.36/images/2022/05/08/202205082105652.png)

上面，我先创建了一个 `test.sh` 文件，然后删除它，再用 `rrm` 恢复文件。其中的 `rm` 和 `rrm` 其实是我刚写的两个脚本的文件别名，后面会讲到。可以看到，`rrm` 可以实现直接恢复文件到原来的位置 —— 就是通过读取原本的 `trashinfo` 文件内部的信息。记得吧，我们的 `trashinfo` 文件记录了文件原始目录。

先创建一个文件，然后再敲代码。我的恢复脚本名是 recover.sh ，感觉还是比较直观。

```bash
$ vim ~/.scripts/recover.sh
```

按照上面的步骤，我们先写好脚本框架，然后再慢慢补充。路径和上面的删除脚本都一样，照搬。

```bash
#!/bin/bash

# TrashPath 表示被删文件的路径
declare -r TrashPath=$HOME/.local/share/Trash/files
# InfoPath 表示信息文件的位置
declare -r InfoPath=$HOME/.local/share/Trash/info

# 确认回收站的位置是否存在
[ ! -e "$TrashPath" ] && mkdir -p "$TrashPath"
[ ! -e "$InfoPath" ] && mkdir -p "$InfoPath"

function recover() {

}

function main() {
  
  while [ $# -gt 0 ]; do
    recover "$1"
    shift 
  done
}

main "$@"
```

接下来只需要补充函数 `recover` 即可。其实这个框架和上一个脚本都差不多 —— 只是上一个脚本没有隔离出新的函数罢了。脚本的使用方法：`rrm path_to_file1 path_to_file2 ...` 。这里的而 `path_to file1` 要求是全路径 —— 也就是必须指定文件路径`~/.local/share/Trash/files/file1`。为什么？其实这是为了避免误恢复，所以只有全路径才能让操作者脑子清醒一点。

### 恢复前操作

恢复之前，我们需要做一些工作，最主要的是判断参数的正确性。后面的代码都写在 `recover` 函数体中。

```bash
function recover() {
  
  # 判断要恢复的文件存在于否，如果不存在直接返回
  if [ ! -e "$1" ]; then
    echo "No such a file or directory: $1"
    return 1
  fi

  # 获取文件的文件名
  local filename="$(basename "$1")"
  # 拼凑文件信息文件的路径
  local infofile="$InfoPath"/"$filename.trashinfo"
  # 判断文件信息文件是否存在，如果不存在则提示恢复者直接手动移动文件到目的目录就可以
  if [ ! -e "$infofile" ]; then
    echo "No such a file: $infofile . Please move target file or directory manually."
    return 1
  fi
}
```

上面的操作我们判断恢复文件的存在和拼凑文件信息文件，为后面的恢复做准备。这里的操作我们其实都见过，例如 `basename` 获取文件名，还有 `-e` 判断文件存在。

### 开始恢复

恢复代码写在上面的代码之后 —— 先看我们的恢复代码：

```bash
local path=""
while read line; do
  ...
  ...
done < "$infofile"
```

首先声明一个本地变量 —— 我不是很喜欢全局变量。这里使用了一个命令 —— `read` 。 `read` 命令语法是：`read variable`。后面的 `variable` 是变量名，可以随便取，我这里的变量名是 `line`。注意，`done` 之后的重定向 —— 其实就是把文件信息文件内的内容一行一行地读出来。重定向我们上面说过了，这就是重定向的用法。这里不停的读取一行一行信息。

```bash
local path=""
while read line; do
  if [[ "$line" =~ "Path" ]]; then
    path="${line:$(expr index "$line" '=')}"
    break
  fi
done < "$infofile"
```

上面的代码就是判断读入的每一行，如果这行中有 “Path” 字样，就进行处理，并退出循环。`=~` 运算符之前也有介绍过，就是判断某一个字符串是否包含另一个字符串。举个例子，我可以使用 `[[ "$str" =~ "chunshu" ]]` 判断 `$str` 是否包含 `chunshu`这个字符串。如果我们读入的行有 "Path" 字样，就说明这就是保存文件路径的一行，后面的命令其实是对这个的处理。

Shell 用 <ruby>切片<rt>Slice</rt></ruby>切割字符串 —— 之前我们介绍的 `#` 和 `%` 就属于 <ruby>切片<rt>Slice</rt></ruby>。这里使用了两个表达式：

1. `${str:from:length}` 。意思是截取某一个字字符串。`str` 表示字符串，`from` 表示从第几个字符开始（不包括这个字符），`length` 表示总共截取多少个字母 —— 可以省略，表示截取到结尾。
2.  `expr index "$str" 'char(s)'` 。 表示获取 `char(s)` 在 `str` 中的下标。`expr` 和 `index` 是保留字，`str` 表示字符串，`char(s)` 表示查找的字符或字符串。

所以 `${line:$(expr index "$line" '=')}` 表示在这一行中找到 `=` 的下标，然后截取 `=` 之后的字符串。如果 `$line` 表示 `Path=/home/chunshuyumao/file1` ，这结果是 `/home/chunshuyumao/file1`。获得路径之后直接使用 `break`跳出 `while`循环。

使用系统的 `/bin/mv` 移动文件到指定位置，使用 `/bin/rm` 删除文件信息文件。

```bash
while read line; do
...
done < "$infofile"

# 一定要使用全路径命令
# Remove From                   To
/bin/mv "$TrashPath/$filename" "$path"
/bin/rm "$infofile"

# 输出提示信息
echo "recover $filename to $path"
```

为什么这里的 `rm` 和 `mv` 要使用全路径（加 `/bin`）而不是直接使用 `rm` 和 `mv`？这是为了避免使用错误的命令。后面会有介绍。赋予脚本可执行权限，然后使用。

```bash
$ chmod u+x ~/.scripts/recover.sh
```

## 使用命令别名

我们最开始的目标是防止自己误删文件导致后悔不已，可是现在的脚本是 mv2trash.sh 和 recover.sh ，我们基本不会使用这种长命令，所以接下来配置命令别名：打开自己的 Shell 配置文件 —— 我用的是 Zsh 所以配置的是 ~/.zshrc ，如果你使用的是 Bash，配置的是应该是 ~/.bashrc 。如果不知道自己的是什么 Shell ，可以输入 `echo $SHELL` ，看输出结果。

我使用 VIM 配置 .zshrc 所以操作是：进入 VIM 后，按 <kbd>Shift</kbd>+<kbd>g</kbd>转到最后一行，然后输入 <kbd>o</kbd> ，输入下面的别名，按 <kbd>Esc</kbd> 键，然后输入 `:wq` ，回车 —— 完成保存退出。在命令行上输入 `source ~/.zshrc` 立即生效。

```bash
$ vim ~/.zshrc

# 这是别名
alias rm=$HOME/.scripts/mv2trash.sh
alias rrm=$HOME/.scripts/recover.sh

$ source ~/.zshrc
```

完成别名，接下来我们使用 的`rm` 就是自己的脚本，`rrm` 是恢复命令。如果想使用系统的删除命令直接删除某个文件而不是挪到垃圾箱怎么办？简单 —— 使用全路径，就是 `/bin/rm file` 。这就是为啥上面的脚本要使用全路径。

# 后话

其实这个脚本还是可写很多，例如我还实现了脚本的日志 —— 不过我看了一下现在字数非常多，没心思再写下去。此外，如果对自己写的脚本不满意，可以使用软件 trash-cli 。这是大佬实现的命令行垃圾箱，不过我没用过，应该比自己写的好一点。很多 Linux 发行版都可以直接安装。使用说明推荐使用 `man trash-cli` 或者 `trash-cli --help` 查看 —— 当然，我没用过，不知道怎么样，自己的够用了。

后面打算写写 VIM 语言 —— 分享分享 VIM 这个被说的神乎其神的编辑器怎么入门。
