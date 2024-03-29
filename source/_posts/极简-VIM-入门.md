---
title: 极简 VIM 入门
hide: false
tags:
  - 编辑器
  - VIM
categories:
  - Linux
math: false
mermaid: false
date: 2022-05-09 18:44:27
index_img:
excerpt: VIM 入门
---

# 前言

<ruby>VIM<rt>/vɪm/</rt></ruby> 是一个古老的编辑器。有人听说这个编辑器可以脱离鼠标、完全依赖键盘操作之后，会好奇地上各种社交、知识问答类网站询问这款编辑器如何。他们通常会得到这样的回答：学习曲线非常陡峭，但学好之后效率非常。  

听到这样的回答，有些新手就开始退缩，而有些人觉得这值得挑战，选择继续练习。于是他们会跑到网上查看相关的 VIM 教程，写教程的人基本都是用过 VIM 或者从不同地方复制过各类教程引流，所以他们上来就先要把这个初学者镇住，以展现自己丰富的知识和 VIM 经验。请看 VIM 的快捷键：

![基础操作](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205091902478.gif)

![进阶操作](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205091902865.png)

就这样，一个初学者还没有入门，基本就被劝退了。这就好比，刚学英语的时候被告知“英语至少有 25 万个单词，历史上堆积的甚至超过 40 万个，一个人一辈子大约 8 万天，要背完所有的单词至少每天记住 5 个单词”，我相信就是丘吉尔也觉得活着好累。所以学 VIM 的时候，要想退出很简单 —— 找个大佬，让他教你，他基本会把你劝退，因为他大概率上来就是一个快捷键图。

秉承着劝退新手的使命，我今天也来讲讲如何学习 VIM。

# VIM 入门

VIM 能够靠键盘操作，其实和它的“语言”有关。想要使用这个编辑器，最简单的方法就是学习它的语言。别吐嘈自己英文不好，VIM 的语法就是“动词 + 名词”这么简单。下面我会分两个部分介绍 VIM：工作模式和语法。工作模式是最基本的概念，如果使用过 VIM，可以直接跳过这一部分。

## 工作模式

VIM 有四种工作模式：

1. <ruby>普通模式<rt>Normal Mode</rt></ruby>（正常模式），这是进入 VIM 后默认的模式，也是其余模式工作的前提。想要进入这个模式很简单：按 <kbd>Esc</kbd> 键即可。
2. <ruby>插入模式<rt>Insert Mode</rt></ruby>，普罗大众眼中编辑器的默认模式。这种模式就是打字时的模式，许多编辑器的默认模式就是这个。
3. <ruby>命令（行）模式<rt>Command Mode</rt></ruby>，这个模式基本用于批量处理和查找、替换，通过输入命令完成操作。
4. <ruby>可视（化）模式<rt>Visual Mode</rt></ruby>，其实就是选择模式。由于 VIM 是键盘操作，所以进行多行选择的时候我们不能通过鼠标操作，这时候就需要借助可视模式。如果你不知道什么叫做多行操作的话，非常好 —— 这说明这个模式你基本用不到。

如果使用 Windows 系统，我还是建议不要靠近 VIM，因为你基本不可能习惯键盘操作。在使用 VIM 之前，我建议在家目录下的 VIM 配置文件 .vimrc 后面添加一些配置。具体使用是：复制下面的设置，在命令行上输入 `vim ~/.vimrc`，回车进入 VIM，输入 <kbd>Shift</kbd>+<kbd>g</kbd>，然后 <kbd>o</kbd>，<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>v</kbd> 粘贴。按 <kbd>Esc</kbd> ，输入 `:wq`，回车。

添加这个配置是修改 VIM 的 <ruby>状态栏<rt>statusline</rt></ruby>，以便后面学习。

```bash
set report=0
set laststatus=2
set statusline=%<%F%m%r%h%=%(%{&fileformat}\ %{&encoding}\ %c,%l\ %p%%%)
```

### 普通模式

普通模式需要记住的操作比较少，记住 —— 普通模式是不可以修改文件和输入内容的！VIM 的普通模式主要用于移动鼠标和视图。很多人习惯使用键盘上的箭头键，但是 VIM 使用的是 <kbd>h</kbd>、 <kbd>j</kbd>、<kbd>k</kbd>、<kbd>l</kbd> 表示左、下、上、右。为什么这么奇葩？因为上古时代的键盘就是这样的（请看下图），那个时候的编辑器还叫 VI （VIM 是 VI 的增强版，VI Improved）。此外，看一下下面键盘的 <kbd>Esc</kbd> 键是在 <kbd>Q</kbd> 左边，所以那个时候按退出键比现在简单。记住这些就可以了。

![上古时代的键盘](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205091946872.jpg)


### 插入模式

插入模式是从普通模式转换。先见过插入命令：

1. i，insert 的第一个字母，表示在光标之前插入；I，insert 大写的第一个字母，表示在这一行的最开始插入。
2. a，append 的第一个字母，表示在光标之后插入；A，append 大写的第一个字母，表示在这一行的最后插入。
3. o，没有对应的英文，表示在光标下一行插入空白行并转到下一行；O，无对应英文，表示在光标上一行添加空白行并撞到上一行。
4. s，substitute 的第一个字母，表示删除光标下的字母并进入插入模式，即替换；S，substitute 大写的第一个字母，表示替换光标所在行。

现在打开一个文件，像我一样通过 <kbd>h</kbd>、<kbd>j</kbd> 、<kbd>k</kbd>、<kbd>l</kbd> 随便选取一个位置。

![移动到指定位置](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205091956771.png)
现在我在 not 之前插入字符，那就把光标移动到 n 上，然后按 <kbd>i</kbd> 键，输入字符，效果如下：

![输入 you](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092009991.png)
可以看到，这时候已经进入插入模式，所以左下角有一个“-- INSERT --” 标志。其余的命令可以自己尝试。

### 命令模式

要进入命令模式先进入普通模式：按 <kbd>Esc</kbd> 。命令模式其实就是普通模式加命令。命令模式的命令以冒号（:）开头，请在输入的时候注意左下角会出现命令 —— 冒号需要打出来。

![命令模式](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092015997.png)

我们需要记住的命令有：

1. `:q!`回车，退出文件不保存。在普通模式下直接输入该命令。`q` 是 quit 单词的缩写。`!` 表示否定，即不保存。
2. `:wq`回车，保存并退出。`w` 是 write 的缩写。 VIM 编辑器打开文件的时候会生成一个 swap 文件，我们的操作都在这个文件，所有操作不会影响原来的文件。如果想要保存，那就把 swap 文件的内容<ruby>写到<rt>write</rt></ruby>原文件，所以使用的是 write 而不是 save。`w` 也可以单独使用。
3. `:s/old/new/`回车，替换。`s` 是 substitute 的缩写。这个操作只会影响光标所在行的第一个匹配项。不常用，经常使用的是全局替换 `:%s/old/new/g` 。`%` 表示所有行，`g` 是 globally 的缩写。`old` 是想要替换的字符串，`new` 是替换后的字符串。

记住这些就行了。第三个命令甚至不用记住 —— 因为一般人不会使用全局替换。

### 可视模式

这个用于选择。在其他编辑器中，我们都是使用鼠标选择，然后滚动滚轮进行选择，在 VIM 中选择需要进入可视模式。进入可视模式之前先确认自己在普通模式：按 <kbd>Esc</kbd> 。进入可视模式的命令有两个：

1. v，visual 的缩写，通常称为行可视化，进行行选择。
2. V，visual 的大写缩写，通常称为块可视化，进行块选择。

比如下面，我要选择第二第三行，使用行可视化，会看到左下角有 “-- VISUAL --”的标志。然后通过 <kbd>h</kbd>、<kbd>j</kbd> 、<kbd>k</kbd>、<kbd>l</kbd> 进行选择。这里使用 <kbd>j</kbd> 向下选择。选择完之后可以按 <kbd>Esc</kbd> 退出。因为还没有教其他操作，所以这里先不进行其他操作。

![选择第二第三行](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092026991.png)

![进入可视模式](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092029374.png)

![按 j 进行向下选择](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092030765.png "按 j 进行向下选择")

下面来看块模式。块模式通常用于插入和删除。比如我要在第一第二第三行之前插入我的大名：光标移动到第一行的第一个字符， <kbd>Shift</kbd>+<kbd>v</kbd>（也就是大写 V），把光标往下移动到第三行，输入 <kbd>Shift</kbd>+<kbd>i</kbd>，然后输入 chunshuyumao —— 注意，这时候只有第一行出现了 chunshuyumao，接下来连按两次 <kbd>Esc</kbd> ，完成。

![Shift + i，输入 chunshuyumao](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092035400.png)

![按两次 Esc 键](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092038342.png)

到这里，最基础的部分就讲完了。开始学习语法部分。

## 语法

VIM 的语法主要是“动词 [+ 介词] + 名词”。介词一般可以省略，但是我下面不会省略介词。

介绍几个常用的动词：

1. d，delete 的缩写，用于删除。
2. y，yank 的缩写，用于复制。这个比较奇葩，因为复制我们都认为是 copy，可是 VIM 用 yank（拖拽）表示复制。
3. c，change 的缩写，用于修改。
4. p，paste 的缩写，用于粘贴。
5. v，visual select 的缩写，用于选择。我们已经见过了，其实就是可视化。
6. r，replace 的缩写，用于替换。
7. x，无对应单词，用于删除光标下的单个字符，但是不会进入插入模式。

第 6 个动词用处不是很大，因为它一般用于修改单个字符；第 7 个只是用于删除单个字符。

名词有：

1. w，word 的缩写，表示一个字。
2. s，sentence 的缩写，表示一个句子。
3. p，paragraph 的缩写，表示一个段落。

介词有：

1. i，inside 的缩写，表示在……内部。
2. a，around 的缩写，表示在……之外。
3. t，to 的缩写，表示到……之前（不包括该字符），数学表达：左闭开 [   )。
4. f，forward 的缩写，表示到……之前（包括该字符），数学表达：左闭右闭 [ ]。 t 和 f 比较难以理解，其实就是一个包含于不包含的关系。

所有的操作都要在普通模式下进行，所以请确认你按了 <kbd>Esc</kbd> 键。

### 动词 + 介词 + 名词

看下图，我现在要删除 important —— delete inside word，缩写就是 diw。使用 inside ，是因为我们的光标在这个单词之内。

![删除之前](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092057496.png "删除之前")

![删除之后](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092059060.png)

删除之后，我们发现 more 和 than 之间的空格有点大，因为 important 被删，空格变成了两个。如果我们想要把空格也删掉，可以在删除 important 的时候使用 delete around word（也就是 daw)。如下：

![把单词和空格一起删掉](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092103676.png "把单词和空格一起删掉")

同理，想要删掉一个句子，使用 delete inside sentence（dis）或者 delete around sentence（das）。还有删除段落: delete inside paragraph（dip）和 delete around paragraph（dap）。

#### 冷语法

除了上面的名词，搞编程的同学可能会碰到这样的问题：删除 main 函数的内容 —— 但是不删除括号。如下，我们需要做的就是 delete inside {（dt{），结果如下：

![删除 main 函数花括号内的内容](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092207065.png)

![效果](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092210408.png)

如果想要把花括号也给删了，那就使用 delete around {（da{）。这个 { 可以改成任何符号，例如 [、( 等等。如果遇到网页使用的<ruby>标签<rt>tag</rt></ruby>，可以使用 t 代替，例如我上面的 `<img>` 标签，我想删掉标签 `<img>` 内的内容就使用 delete inside tag（dit），删除内容和标签就使用 delete around tag（dat)。


### 动词 + 数量词 + 介词 + 名词

如果我想删除两个单词呢？删除两个单词用英语怎么说：delete 2 inside word（d2iw）或者 delete 2 around word（d2aw）。注意，如果你使用 inside ，空格也会被当作一个单词，所以建议使用 around。

删除两行？delete 2 inside paragraph（d2ip）或者 delete 2 around paragraph。

修改单词：change inside word（ciw）或者change around word（caw）；修改两个单词：change 2 inseide word（c2iw）或者 change 2 around word（c2aw）。

上面的介词都可以省略 —— 注意，省略的后果是：不论是句子、单词还是段落，编辑器默认从光标开始。例如使用 delete 2 word（d2w）：编辑器就认为你所说的两个单词是从光标之后。看下面的例子

![删除之前](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092057496.png "删除之前")

![不使用介词,，输入 d2w](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092116759.png "不使用介词,，输入 d2w")

### 惯用法

现在将光标移动到第九行，输入 yank inside sentence（yis）复制一行。输入 paste（p）就会粘贴到下一行（大写 P 会粘贴到上一行）。

![复制一行](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092120488.png)

![粘贴](http://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2022/05/09/202205092123546.png)

虽然听起来很霸气，但是我们删除一行或者复制一行通常不会使用 dis（das）或者 yis（yas），因为需要三个字母 —— 我们是懒人，单词好越少越好，所以一般使用 dd 或者 yy 进行删除和复制。类似的还有使用 cc 代替 change inside sentence（cis）。

### 数量词 + 动词 + 名词

这个和 “动词 + 数量词 + 介词 + 名词” 差不多，但是一般用于特殊的动词，例如 r(replace)、x(删除单个字符)、p(paste)。这些命令一般都是单个使用 —— 不存在 paste inside sentence 这个类用法，所以要粘贴多次的使用可以使用 2 paste（读作 twice paste）。

此外还可以搭配 yank 使用 —— yank 2 around sentence（y2as）= 2 yank yank（2yy），这样要简单很多。还有删除 d 等等。基本是更简单的用法。当然，其实“动词 + 数量词 + 介词 + 名词”也可以省略为“动词 + 数量词 + 名词”。

### 其他动词

上面列出的是常用的动词，这里还有一些补充的动词：

1. u，undo 的缩写，用于撤销操作，适用于上面的语法，可以单独使用，也可以搭配数量词（不需要介词）。
2. <kbd>Ctrl</kbd>+<kbd>r</kbd>，redo 的缩写，不适用于上面的语法，单独使用，算作快捷键。撤销 撤销操作（我没有多写一个撤销）。
3. g，goto 的缩写，用于跳转到指定行。适用于上面的语法，不需要介词。gg 表示跳转到第一行，<kbd>Shift</kbd>+<kbd>g</kbd> 表示跳转到最后一行。跳转到 45 行使用 45gg。
4. z，无对应单词，表示滚动当前行。zz 表示滚动当前行到屏幕中间；zb，b 是 bottom 的缩写，表示滚动当前行到屏幕底部；zt，t 是 top 的缩写，表示滚动当前行到屏幕顶部。
5. w，word 的缩写，做动词表示移动到下一个单词开头。2w 表示移动到后面第二个单词的开头。适用与上面的部分语法。
6. e，end 的缩写，做动词表示移动到单词的结尾，与 b 相反。适用于上面的部分语法。
7. b，begin 的缩写，做动词表示移动到单词的开头，与 e 相反。是用于上面的部分语法。
8. f，find 的缩写，表示查找。ft 表示查找该行的字符 t ，并移动到相应位置。
9. ^，head of line 的符号，表示开头，做动词表示移动到该行的开头，与 $ 相反。配合 delete 使用可以做到从当前字符删除到行首。可以当作名词与大部分动词搭配使用。
10. $，end of line 的符号，表示结尾，做动词表示移动到该行的结尾，与 ^ 相反。可以作为名词与大部分动词搭配使用。

# 结语

稍微介绍了一下 VIM 的语法，其实完全不算难。比如 b、e、f、w 我就很少用。用的多的基本是 goto、delete、insert、yank 和 paste。这里的功能基本是我用到的部分，其余如快捷键映射、宏录制甚至 VIM 脚本语法都是进阶用法（意思是说，我也未必知道），我们就懒得涉猎了。

祝 VIM 入门愉快！