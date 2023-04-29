---
title: 在 VIM 中安装 vim-pandoc-syntax 插件
hide: false
tags:
  - VIM
  - Pandoc
categories:
  - VIM
math: false
mermaid: false
date: 2023-04-29 18:43:40
---

# 前言

[vim-pandoc-syntax](https://github.com/vim-pandoc/vim-pandoc-syntax "vim-pandoc-syntax GitHub 仓库") 是 [VIM](https://www.vim.org "VIM") 编辑器的 Pandoc 语法插件，支持大部分的 Pandoc Markdown 扩展（Pandoc's Markdown）语法特性。简单来说就是高亮。口说无凭，下面可以看看 vim-pandoc-syntax 的高亮功能：

![不开启语法高亮](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429192542.png)

![开启 vim-pandoc-syntax 高亮功能](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429192005.png)

可以看到确实漂亮，特别是用上了 VIM 的 conceal 特性，可以使用很多的 Unicode 字符代替 ASCII 字符。

很久之前就听说了 vim-pandoc-syntax 插件，但一直没有尝试，最近有时间试试。

# 安装插件

安装插件之前最好确认自己用的是 VIM 编辑器，因为这是 VIM 插件。

## 安装

首先是到 [Github](https://github.com/vim-pandoc/vim-pandoc-syntax "vim-pandoc-syntax GitHub 仓库") 下载 vim-pandoc-syntax 插件。因为很多人都使用插件管理器，可以按照 Github 上的教程直接安装。我不用什么管理器，手动安装。

首先确认自己 VIM 有 pack 文件夹，控制台上输入：

```sh
$ file ~/.vim/pack/plugins/start
/home/chunshuyumao/.vim/pack/plugins/start: directory
```

如果输出结果不是上面的，而是类似下面的提示，说明自己的 VIM 并没有这个路径。

```sh
/home/chunshuyumao/.vim/pack/plugins/start: cannot open `/home/chunshuyumao/.vim/pack/plugins/start' (No such file or directory)
```

>
> 上面的 plugins 其实可以是任何名称，不是中文就行。例如:
> 
> /home/chunshuyumao/.vim/pack/anything/start
> 

建立一个文件夹，在控制台上输入以下命令：

```sh
$ mkdir -vp ~/.vim/pack/plugins/start
mkdir: created directory '/home/chunshuyumao/.vim/pack/plugins/start'
```

建立文件夹之后，进入该文件夹。用 [Git](https://git-scm.com "Git") 克隆（clone）仓库或者直接去 GitHub 上下载 vim-pandoc-syntax 压缩包，解压后放到刚刚创建的文件夹。如下面的操作：

```sh
$ cd ~/.vim/pack/plugins/start
$ pwd
/home/chunshuyumao/.vim/pack/plugins/start
$ git clone https://github.com/vim-pandoc/vim-pandoc-syntax
Cloning into 'vim-pandoc-syntax'...
...
$ ls
vim-pandoc-syntax
```

在 VIM 的配置文件 `~/.vimrc` 中添加：

```vim
augroup pandoc_syntax
    au! BufNewFile,BufFilePre,BufRead *.md set filetype=markdown.pandoc
augroup END
```

保存退出即可，这就算安装好了。

打开 VIM，然后进入命令行模式，即输入 `:` 进入命令行模式。输入：

```sh
:helptag ~/.vim/pack/plugins/start/vim-pandoc-syntax/doc
```

生成 vim-pandoc-syntax 插件的说明文档。这样自己忘了关于 vim-pandoc-syntax 的配置时，可以直接在 VIM 命令行模式查找帮助。

进入命令行模式输入 `:help vim-pandoc-syntax` 查看帮助文件是否正确生成。

![在命令行模式查看 vim-pandoc-syntax 帮助文档](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429195431.gif) 

## 配置

今天写这个博客肯定不是单单安装就完了，还有一些需要配置。

### 代码块高亮

vim-pandoc-syntax 可使 Markdown 代码块高亮，但默认不开启，可以使用 `:help pandoc-syntax-commands` 查看帮助。

![代码块高亮的帮助，文档表明可以使用 `g:pandoc#syntax#codeblocks#enable#langs` 或者 `:PandocHighlight` 手动调用高亮](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429200304.png)

按照帮助，我们可以在 `g:pandoc#syntax#codeblocks#enable#langs` 中列出自己的需要高亮的语言，或者使用 `:PandocHighlight LANG` 高亮，其中 `LANG` 是需要高亮的语言的名称，如 `cpp` 是 C++ ，`python` 是 Python。 

先说手动高亮。手动高亮就是在自己写 Markdown 时，用到什么语言再调用高亮。调用方式是使用 VIM 的命令行模式，输入上面说的命令。下面的动图展示如何调用语言高亮。请仔细看，在调用高亮之前 Markdown 文档的 VIM 代码块是没有高亮的，只显示了淡蓝色。

![手动调用语法高亮](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429200812.gif)

这种方式比较粗糙，如果嫌麻烦，可以使用自动高亮，也就是下面要讲的。

要自动高亮需要在 VIM 的配置文件 `~/.vimrc` 中添加：

```vim
let g:pandoc#syntax#codeblocks#enable#langs = [
      \ 'cpp',
      \ 'python',
      \ 'sh',
      \ 'bash',
      \ 'lua',
      \ 'vim',
      \ ]
``` 

上面是我可能用到的几种语言。语言不是写得越多越好，毕竟有些语言自己也用不到，写下来作甚？写完之后保存退出。这样只要自己在代码块上标注使用了什么语言，VIM 就会自动帮助你高亮。其他的配置也可以看文档。

### 与 polyglot 插件冲突

我的 VIM 安装的插件不多，但是 vim-pandoc-syntax 还是和其他插件冲突了。我使用 [vim-polyglot](https://github.com/sheerun/vim-polyglot "vim-polyglot 官网") 作为自己的语法高亮包。这是一个集合了很多语言和文件高亮方式的插件包，Markdown 自然也包含其中。如果没有安装这个插件，可以不用看关于这个插件冲突的解决方法。

首先查看帮助文档：

```vim
:help poly-vim
```

![文档说可以使用 `g:polyglot_disabled` 关闭特定语言的高亮](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429202454.png)

其实这个文档说的不够详细，可以参考更详细的 vim-polyglot [README.md](https://github.com/sheerun/vim-polyglot/blob/master/README.md "vim-polyglot README.md 文档") 。

在 `~/.vimrc` 中加入：

```vim
let g:polyglot_disabled = [
    \ 'markdown',
    \ ]
```

来禁止 vim-polyglot 帮我们高亮自己的 Markdown 文件，因为使用 vim-pandoc-syntax 效果更好。

vim-polyglot 除了可以给我们的 Markdown 文件语法高亮，附带了一个可以查看、生成和跳转目录的插件，配置在 `vim-polyglot/ftplugin` 下的 `markdown.vim` 文件。

![vim-polyglot 带的 Markdown 目录包生成的目录](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429203428.png)

我们希望使用这个可以生成目录的文件，但又不想使用 vim-polyglot 带的高亮。vim-polyglot 提供了方法，只是帮助文档没有写，上面提到的 README.md 文件里边写了，就是把上面添加的 `g:polyglot_disabled` 那一行改成：

```vim
let g:polyglot_disabled = [
    \ 'markdown.plugin',
    \ ]
```

可以了。我们可以看看是不是可以了。

![可以了！但是好像有点不同](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429204013.png)

可以但有点不同，可以参考上面的一张图，好像代码块的边边不一样了：之前是一个 λ 和语言名称，现在不见了。其实不是不见了，而是这个 `ftplugin` 下的文件也同样对代码块有自己的想法。这样设计不好，一个插件应该只做一件事，它同时负责了代码块高亮和目录生成。

我们只要可以生成目录就行，不需要高亮，高亮已经有负责的插件了。直接打开 `vim-polyglot` 文件夹下 `ftplugin` 文件夹内的 `markdown.vim` 文件，查看是哪几行在作怪。 

![谁在作怪？](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429204516.png)

可以看到，从 774 行之后都是关于语法高亮的部分 —— 我自然不可能只看这一点点就看出来了，我早就把文件看完了才这么肯定是这几行作怪的。要想去掉这个功能，只需要全删掉 774 行之后的代码，或者直接注释掉。我的习惯是注释掉，因为搞不好以后还有用，所以直接用 `"` 注释吧！

使用 VIM 就不要一个一个插入注释符号了，使用命令行模式，输入 `:774,$s/^/"/` 进行注释，回车即可。

![命令行直接替换](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429205440.gif) 

- `774,$` 是选择第 774 行到最后一行（`$`代表最后一行）
- `s` 是 `substitute`，”替换“的缩写
- `/` 用做分隔符
- `^` 表示是一行开头的字符
- `/` 还是分隔符
- `"` 是替换开头的字符
- `/` 还是分隔符

### 改进 vim-pandoc-syntax 插件

vim-pandoc-syntax 插件好是好，但有一个问题：斜体必须前后都是空格。

举个例子，要给 `you silly man` 中的 `silly` 斜体必须要这样写 `you *silly* man`。这么说大家可能觉得没啥，因为这很明显呀，难不成我还想写成 `you*silly*man`？其实不然，这是英文大家才觉得正常，中文字与字之间是没有空格的。

![没有空格，即使是英文也显示不出斜体](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429211017.png)

为了让它可以在没有空格和中文环境下也可以体现斜体，我们要做一些改变。当然，有些人觉得这没必要，因为中文一般不会使用斜体，这种修改多此一举。那我举个英文的例子好了：

生物领域里，限制性核酸内切酶由三个斜体字母命名，其后可以添加一个分离菌株的正体缩写，例如 *Eco*RⅠ 酶、*Bam*HⅠ 酶。这种时候即使是英文也没有空格。生物是一个严肃的学科，书写是专业性的体现，这种情况总是必要的吧？如果觉得没必要，那最好不好进实验室，不然会被导师骂死。

截个图看看没有空格之后的格式多混乱：

![没有空格的混乱格式](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429211751.png)

没有空格导致后面的文字都被当成了斜体的内容，VIM 在等后接一个空格的 `*` 作为斜体的结束标志。等不它自然觉得都是斜体咯！

闲话少叙。不用找斜体在哪了，我已经找到了：vim-pandoc-syntax 插件目录内，syntax 目录下的 `pandoc.vim` 文件的第 352、353 行。说实话，这正则我看了都头疼。

![斜体部分的正则](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429212547.png)

这个正则非常复杂，我就不解释了 —— 其实也不全看懂，可以在 VIM 内输入 `:help regexp` 查看，如果想看这些鬼东西代表什么的话。

主要看看 `start` 和 `end` 两部分内的鬼东西就行了。这里我们看有个 `\s`，这是空白字符的意思，这里就定义了必须要碰到起码一个空白字符。现在我们需要定义的是，是不是空白都可以，所以在后面添加 `\S`，这表示非空白字符。

```vim
start=/\\\@1<!\(\_^\|\s\|[[:punct:]]\)\@<=\*\S\@=/ skip=/\(\*\*\|__\)/ end=/\*\([[:punct:]]\|\s\|\_$\)\@=/
start=/\\\@1<!\(\_^\|\s\|[[:punct:]]\)\@<=_\S\@=/ skip=/\(\*\*\|__\)/ end=/\S\@1<=_\([[:punct:]]\|\s\|\_$\)\@=/
```

现在这两行变成：

```vim
" 把 \s 换成 \s\|\S，一行改两处，总四处
"start=/\\\@1<!\(\_^\|\s\|[[:punct:]]\)\@<=\*\S\@=/ skip=/\(\*\*\|__\)/ end=/\*\([[:punct:]]\|\s\|\_$\)\@=/
"                       ↓                                                                         ↓
start=/\\\@1<!\(\_^\|\s\|\S\|[[:punct:]]\)\@<=\*\S\@=/ skip=/\(\*\*\|__\)/ end=/\*\([[:punct:]]\|\s\|\S\|\_$\)\@=/
"start=/\\\@1<!\(\_^\|\s\|[[:punct:]]\)\@<=_\S\@=/ skip=/\(\*\*\|__\)/ end=/\S\@1<=_\([[:punct:]]\|\s\|\_$\)\@=/
"                       ↓                                                                             ↓
start=/\\\@1<!\(\_^\|\s\|\S\|[[:punct:]]\)\@<=_\S\@=/ skip=/\(\*\*\|__\)/ end=/\S\@1<=_\([[:punct:]]\|\s\S\|\|\_$\)\@=/
```

![修改 `\s` 为 `\s\|\S`](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429214018.png)

![修改之后斜体啦](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429213646.png)

![限制性核酸内切酶也斜体了](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230429213722.png)

大功告成！至此，我觉得这个插件已经被改得比较好了，哈哈。

## 书写规范

之前我写过一篇如何使用 Pandoc 写论文的博客，里边讲到了怎么给表格添加标题：

```markdown

查看[@tbl:id]。

|<++>|<++>|<++>|
|:----:|:----:|:----:|
|<++>|<++>|<++>|
|<++>|<++>|<++>|
|<++>|<++>|<++>|
: 这是标题 {#tbl:id}
```

其实不全对，应该是这样的：

```markdown

查看[@tbl:id]。

|<++>|<++>|<++>|
|:----:|:----:|:----:|
|<++>|<++>|<++>|
|<++>|<++>|<++>|
|<++>|<++>|<++>|

: 这是标题 {#tbl:id}
```

即，表格和标题应该空一行。之前没有空是因为那时使用 [Zettlr](https://zettlr.com "Zettlr")，空一行编辑器就渲染成了新的一行。

这不算大问题，因为 Pandoc 都会渲染成表格标题，所以我一直没说。然而现在推荐了 vim-pandoc-syntax 插件，这个插件认为表格和标题之间是有空行的，如果表格和标题不空一行，它就不进行渲染 —— 表格之后都不进行渲染。故此才得提一下。

# 后言

有阵子没看 [Pandoc](https://pandoc.org "Pandoc") 的更新，现在都到第三个大版本了。Arch Linux 官方仓库的 Pandoc 有很大的依赖，十几 M 的 Pandoc 安装的 [Haskell](https://www.haskell "Haskell Language") 包有 500M 左右。

我是个有洁癖的人，最看不惯这种无所谓的依赖，所以自己安装的 Pandoc 是 [Github 仓库](https://github.com/jpm/pandoc/releases)下载的独立可执行文件，因此才没有跟进 Pandoc 的升级。现在自己使用的还是 2.9 版的 Pandoc。最近看了看，Pandoc 3.0 的变化还是比较大的。

虽然 Pandoc 版本变化大，但是对我这种之用 Markdown 转到 WORD 或者 HTML 的人来说，变化不大：HTML 增加了一些自己不是很常用的功能，WORD 就添加了摘要标题（Abstract Title），作用不大。

作为一个懒散的人，只要目前的 Pandoc 版本足够使用，我就不会改用其他的版本，所以继续停留在 v2.9 也不是不行。
