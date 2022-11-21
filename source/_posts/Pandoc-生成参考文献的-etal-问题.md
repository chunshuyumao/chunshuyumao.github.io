---
title: Pandoc 生成参考文献的 etal 问题
hide: false
tags:
  - Markdown
  - CSL
  - Pandoc Filter
categories:
  - Pandoc
math: false
mermaid: false
date: 2022-11-21 10:26:14
---

# 前言

之前埋了坑，今天得填。使用 Pandoc 按照 CSL(Citation Style Language) 生成参考文献时会因为本地化问题出现一些小问题。

![问题再现](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121113010.png)

CSL 文件可以设置多个本地化判断，但是 Pandoc 渲染出来的依旧是中文。可能和 Zotero 导出的语言设置有关。反正单单使用 CSL 文件改变不了什么，我们需要其他方法。

参考文献输出结果有问题，我们可以考虑用正则表达式进行替换。这种方法最简单，也是多数人能够想到的。此外，Pandoc 允许通过筛选脚本(Filter)对文件进行转换前、后的修改。{% post_link Pandoc-从-Markdown-到其他格式 %} 里引入的 Pandoc-crossref 本质也是一个筛选脚本：筛选出交叉引用的标记并进行格式转化。Pandoc 的命令行 `--citeproc` 选项，未并入 Pandoc 内置命令之前也是一个筛选脚本，现在的 Pandoc 还保留着它的原始用法，即 `--filter pandoc-citeproc`。因此，对 Pandoc 文本进行修改也可以借助筛选脚本。这样我们就有了计划，准备行动吧！

# 正则替换

正则替换有两个办法，一个是在 Word 软件，例如 WPS Word 或者 Microsoft Word 中直接搜索替换。另一个比较复杂需要其他操作，待会解释。

## WPS 正则替换

WPS 可以替换但有点不大合适，如果参考文献的数字标号是括号而不是方括号，我们就得正则表达式，而且这个正则用起来总感觉差点意思，也许叫做通配符好一点。Microsoft Word 也差不多，效果基本如此。此外，这是在 WPS 2019 for linux 测试的，使用不了网上说的 `^s` `^w` 等空白符号，空白符原原本本就是 ` ` 。

```bash
查找：(\[[0-9]{1,}\][ ]*^t{1,}[a-zA-Z, ]{1,})等
替换：\1et al
```

![WPS Word 正则替换 国标2005 的参考文献](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121135756.gif)

 `\[` 和 `\]` 包起来的是参考文献的标号，标号是数字所以用 `[0-9]` 代替数字，`{1,}` 表示数字至少有一个。`[ ]*` 表示任意多个(0 个以上)空白符。空白符可以直接使用 ` ` 替代，但是无法保证只有一个空白符，所以应该选择任意多个。WPS 不支持 ` *` 这种写法，所以使用 `[]` 包住空白符，然后用 `*` 限定。`[]` 表示方框中任何一个符号，比如 `[34]` 表示可以是 `3` 或 `4`。`^t` 是制表符，就是 `Tab` 键，起码一个。`[a-zA-Z, ]` 表示任何英文字母、空白符和逗号。括号括起来表示捕捉这个字符串，后面的“等”表明这个字符串后面必须跟一个中文“等”。“替换”的 `\1` 代表查找到的字符串。这属于硬编码，健壮性不足。

## 纯正的正则

除了 WPS 和 Office 内的替换，我们还可以使用更为神奇的正则，毕竟真正的正则替换编写的代码更健壮一点。不过这一步需要使用支持正则的工具，比如 Python 和 SED。如果电脑本来就装有前者还好，没有的话纯当作长见识。SED 是 Stream EDitor 的缩写，中文叫做流式编辑器。这东东虽然是命令行操作，但本质是一个编辑器。Windows 系统就不用想了，SED 是 Linux 系统标配。

如果使用 Windows，不建议下面的操作，所以下面的演示就当作是 Linux 系统独占——虽然并非如此，但 Windows 的 Dos 真的太弱了，不适合写脚本——不适合我写。

之前说过，DOCX 是二进制文件，但没说是哪一类。DOCX 不是我们理解中的那种二进制，而是一种压缩包，解压缩就能看到内容。

![解压 DOCX 文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121144133.gif)

解压后有一堆文件，都是 XML 文本文件，可以直接使用 SED 进行正则替换。文字内容在 `word/document.xml` 这个文件里。打开文件可以看到内容。

![文字内容位置](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121144416.png)

![文字内容](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121144723.png)

接下来就是替换。使用 SED 暴力替换：

```bash
$ sed -i 's/\([a-zA-Z\.\s]\+\)\(,\s\|\s\)\+等\./\1\2et al./g' word/document.xml
```

`-i` 是原位替换，直接在源文件替换。替换分为三个部分：`s/find/replace/flag`。`s` 表示替换 substitute，`find` 表示目标字符串，`replace` 表示替换后的字符串，`g` 表示全局替换 global，斜杠用来分隔。`\([a-zA-Z\.\s]\+\)\(,\s\|\s\)\+等` 找到字母、逗号、点和空白(`\s`)字符，紧接着一个中文 `等`，然后替换成 `et al`。效果与 WPS 差不多，只是不需要使用 WPS 或者 Office，用 SED 即可。如果替换 HTML，可以直接使用 `sed` 操作即可，如：

```bash
$ sed -i 's/\([a-zA-Z\.\s]\+\)\(,\s\|\s\)\+等\./\1\2et al./g' test.html
```

替换完毕，将这些东东原原本本压缩回去：

```bash
$ zip -r new.docx *
```

![重新压缩](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121145806.png)

用 WPS 打开新文件看看替换的效果：

![替换后的 new.docx 文件](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121150031.png)

替换这一步可以用其他语言写，例如 Python、Lua 等。压缩完毕后只需要覆盖原文件就可以了。这样太麻烦了，可以写一个脚本：

```bash
set -Eeuo pipefail

# 退出时删除临时文件
trap clean EXIT
# 生成临时文件
declare -r tmpdir="$(mktemp -u -d)"
# 获取文件的全路径
declare -r target="$(readlink -m "$1")"

function clean() {
  if [ -e "$tmpdir" ]; then
    /bin/rm -rf "$tmpdir"
  fi
}

function main() {

  # 解压缩，-q 表示 quiet，不用输出信息
  unzip -qd "$tmpdir" "$1"
  pushd "$tmpdir"
  sed -i 's/\([a-zA-Z\.\s]\+\)\(,\s\|\s\)\+等\./\1\2et al./g' word/document.xml
  zip -qr "$target" *
  popd
}

main "$@" > /dev/null
```

保存到某个路径，例如 `/home/chunshuyumao/Documents/Pandoc`，并在 Shell 配置文件，如 `.bashrc` `.zshrc`，加入 `alias reorg=$HOME/chunshuyumao/Documents/Pandoc/reorg.sh`。重新打开终端，使用 `reorg 文件` 就可以完成整个替换。

相信正则替换看着已经头大了。正则替换确实不是很好。举个例子，如果不是生成 DOCX 或者 HTML 而是 PDF 文件，那上帝来了也救不了。相比于 DOCX 这种假的二进制文件，PDF 可是实打实的难以修改，正则只能对付对付文本文件，复杂的二进制只能见鬼。由此可见，简单使用正则不是解决之道。要想彻底解决这个问题，我们还是得回到 Pandoc 这个“罪魁祸首”。

# 解铃还需系铃人

兜了一圈还是回到了 Pandoc。Pandoc 使用 pandoc-citeproc/citeproc 解析参考文献是内部机制改变不了，只能想其他办法。之前说过 Pandoc 支持使用 Lua 写筛选脚本，最好到 [官网](https://pandoc.org/lua-filters.html) 了解 Pandoc 调用筛选脚本的步骤。

```
source -> AST -> filter -> AST -> target
```

![](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121152414.png)

AST 是 Abstract Syntax Tree，抽象语法树的缩写。这东东听起来有点抽象，其实就是给字符加上一点标记。举个例子，`2022` 是一个字符串，如果在 Python 中它被认为是数字，我们就给它一个结构，长这样：

```
struct {
  text = '2022'
  type = number
}
```

这样程序读取的时候只要查看 `type` 就知道，原来这是一个数字。如果 `type` 换成 `string` 那就被认为是字符串。但是内容 `2022` 完全一样。编程语言就是通过这样的方式区分各种标识符到底是什么东西。这东西还可以分析语言，比如 `牛` 这个字，用 `形` 或者 `adj` 表示形容词，用 `名` 或者 `n` 表示名词。但是 `牛` 完全就一个写法。机器就是通过这个笨拙的方式去了解语言。

我解释这个是为啥？因为无聊。

写 Filter，我们解决的是 Pandoc 读取所有源文件之后的数据，处理完之后 Pandoc 再进行处理输出。废话不多说，开始干活。

## Pandoc 抽象语法树

我们得先了解这个语法树是个什么东西，Pandoc 提供了一个叫 `native` 的格式允许我们将自己的格式转换成它的语法文件。打开命令行，输入

```bash
$ pandoc -d ~/Documents/Pandoc/defaults/HTML.yaml -t native test.md -o test.json
```

YAML 文件是上一篇文章 {% post_link Pandoc-从-Markdown-到其他格式%} 最后写的文件，调用 Pandoc-crossref、citeproc 和一些其他的配置。这个 test.md 文件只需要包含引用即可，没什么别的要求。native 格式不一定是 JSON，我只是为了高亮显示，所以后缀写成了 JSON。转换结束后打开：

![红框内为语法标记](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121154858.png)

![一篇文献在 Pandoc 内的表示](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121155403.png)

可以看到，Pandoc 内部使用一个 `Div` 标识包住一篇参考文献，而且标签中有 `ref`字样，参考文献内有一个奇葩的字符串 `\31561` 代表中文的“等”，类型标识是 `Str`。记住这两个即可。

## 编写 Pandoc Filter

Pandoc Filter 的说明还是蛮清晰的，这里直接使用，不进行讲解。Pandoc 解析输入文件生成的语法树分为两部分，一部分叫做 `Pandoc` 代表正文，另一部分是 `Meta` 表示 `front-matter` 的信息。我们要修改正文，所以只需处理 Pandoc。在之前的目录 `/home/chunshuyumao/Documents/Pandoc` 新建一个 filters 目录，目录里边创建一个 cite_et_al.lua 文件。

```bash
$ mkdir -p /home/chunshuyumao/Documents/Pandoc/filters
$ touch cite_et_al.lua
```

打开 cite_et_al.lua，在里边写入以下内容：

```lua
function Pandoc(doc)
  doc:walk{
    Div = function(div)
      print(div)
    end,
  }
end
```

只处理 Pandoc 部分所以创建一个 Pandoc 函数，文档中指出 Pandoc 函数只有一个参数，取名为 doc. 几乎所有的 Pandoc 类型都有一个 `walk` 函数用于遍历其内部元素，所以调用它，传入的参数是一个 Table(表)，这是 Lua 内最常用的数据类型，与 Python 的表 `{}` 差不多。表内有唯一的一个 key(键) 叫做 `Div`，表示我们只处理 `Div` 类型，这个键对应的 value(值) 是一个函数，函数只是打印 `Div` 类型，其他什么也不做。

Lua 有一个语法糖：当函数只有一个参数时，函数调用的括号可以省略。`walk` 只有一个参数，所以括号被省略，原本的调用应该是:

```lua
doc:walk({
  Div = function(div)
    print(div)
  end,
})
```

`walk` 是 `doc` 的成员，Lua 调用成员函数使用的是 `:`，有别于 Python、类C 语言的 `.`。

好的，解释完毕。打开之前 HTML.yaml 文件，在 filters 选项下添加脚本，并执行转换。

```yaml
filters:
  - pandoc-crossref
  - citeproc
  - cite_et_al.lua
```

![](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121162559.png)

```bash
$ pandoc -d ~/Documents/Pandoc/defaults/HTML.yaml test.md -o test.html
```

下面执行转化不再列出代码，因为输入的就是上面这一行。查看输出：

![意外发现](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121163016.png)

执行后发现很多的 Div 类型被打印，其中有一个类型，即上图红圈内，包含了所有的引用文献。它的标签也很特别，其他文献引用是 “ref-” + citationkey, 它是 “refs”。这就好办了，我们只需要修改它就行了，不用一个一个找其他的文献在哪。它内部的文献仍然是一个 Div 包围。所以我们筛选出来:

```lua
function Pandoc(doc)
  doc:walk{
    Div = function(div)
      -- 查找 refs Div
      if (div.identifier or ''):find('refs') then
        print(div)
      end
    end,
  }
end
```

这个 `identifier` 是 Div 的标签，其实是 Div 里的 attr 内的 identifier 的引用。什么意思内，就是 `div.identifier` 等于 `div.attr.identifier`，因此不必写那么长，可以用 `div.identifier` 代替即可。`or` 表示如果没有 `identifier` 域则用空字符串('')代替，这是 Lua 的语法糖。

![Div 的 attr 结构](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121163956.png)

![attr 内的 identifier](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121164035.png)

执行转化，查看输出：

![成功输出 refs Div](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121164303.png)

找到之后对 refc Div 内的文献操作需要写一个函数：

```lua

-- 这是处理文献的函数
local function ref_div(div)
  
end

function Pandoc(doc)
  return doc:walk{
    Div = function(div)
      -- 查找 refs Div
      if (div.identifier or ''):find('refs') then
        div.content = div.content:walk{ Div = ref_div, }
        return div
      end
    end,
  }
end
```

Div 的内容部分在 `content` 域里，所以我们使用 `walk` 函数对 refs Div 的 content 结构域进行遍历。同时写一个新的函数，函数前面有个 `local` 表示脚本之外无法调用。这是 Lua 的一些技巧，方便索引。遍历结束后返回 content 替换掉 Div 原本的内容，最后返回修改的 Div.

## 处理 Str 部分

记得最开始的分析吧，我们要处理的是文献 Div 里的 Str 类型。因此修改代码：

```lua
local function ref_div(div)
  local stop = false
  local is_en = false

  local function do_subs(str)
  end

  div.content = div.content:walk{ Str = do_subs, }

  return div
end
```

`do_subs` 是 do substitution 的缩写，实现替换。定义在 ref_div 这个函数里边，以便使用上边的两个标志。`ref_div` 处理一篇文献只需要替换一次“等”，所以设置一个标识位 `stop` 表示替换停止。只要替换停止，函数直接返回不再进行替换。`is_en` 判断是不是英文文献，如果是则替换。替换结束后需要把值返回给 div.content，然后再返回 div。

最后编写 do_subs 函数，如下：

```lua
local function do_subs(str)
  if stop then return end

    ---@type string
    local text = str.text
    if not is_en then
      if text:find('%a+') then is_en = true
      elseif text:find('等.') then stop = true end
      return
    end

    local times
    text, times = text:gsub('等', 'et al')
    if times >= 1 then
      str.text = text
      stop = true
      return str
    end
end
```

1. 首先判断是不是停止替换，如果是，直接返回；如果不是，则进行后边的操作。

2. 获取 str 里的字符串，字符串保存在 text 里——可通过 Pandoc 文档查看。判断是不是英文文献，刚开始我们设置不是英文文献，所以它就进行搜索。

3. 使用正则查找英文单词，只要出现英文单词就是英文文献。正则用 `%a+` 来表示至少一个字母，是英文文献就把 `is_en` 设置为 `true`。如果找不到英文单词，不一定不是英文文献，例如可能是 `[1]` 这类的标号。因此再判断一下有没有出现 `等.` 这个中文字符。“等”只出现在中文和英文文献里，前面找不到英文单词，现在又出现“等”，那就铁定是中文文献，所以设置 `stop`，不需要白费功夫替换中文，节省点时间。如果还是没有找到“等”，就直接返回，什么也不要设置。

4. 如果是英文文献，使用 `gsub` 进行替换。返回值分别是替换后的字符和替换次数。替换次数大于等于一说明至少替换了一次，这时可以停止替换，一篇文献不可能出现两次“等”，同时将替换后的字符替换原来的字符并返回。

经过以上四步，文献就替换完成了。验证一下：

![替换成功](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121172624.png)

大功告成！

### 重新优化

查看 [文档](https://pandoc.org/lua-filters.html#module-pandoc.utils) 发现 Pandoc 提供一个 `pandoc.utils.citeproc` 函数进行参考文献解析。我们可以应该到自己的代码中，于是代码变成：

```lua
local function ref_div(div)
  local stop = false
  local is_en = false

  local function do_subs(str)
    if stop then return end

    ---@type string
    local text = str.text
    if not is_en then
      if text:find('%a+') then is_en = true
      elseif text:find('等.') then stop = true end
      return
    end

    local times
    text, times = text:gsub('等', 'et al')
    if times >= 1 then
      str.text = text
      stop = true
      return str
    end
  end

  div.content = div.content:walk{ Str = do_subs, }

  return div
end

function Pandoc(doc)
  return pandoc.utils.citeproc(doc):walk{
    Div = function(div)
      -- 查找 refs Div
      if (div.identifier or ''):find('refs') then
        div.content = div.content:walk{ Div = ref_div, }
        return div
      end
    end,
  }
end
```

HTML.yaml 改为

![不需要 citeproc 了](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121173048.png)

因为在自己的代码中调用了 citeproc，所以不需要在 HTML.yaml 中调用。同理 docx.yaml 也需要改：

![docx.yaml](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/20221121173211.png)

至此，我们写了一个 Lua 脚本用于修改文献。这从根本改变了 Pandoc 的输出，因此即使输出文件是 PDF 也是正确的。这种方法算比较完善的方法了。好煎熬。

# 后语

Pandoc 是一个强大的工具。然而这个世界没有什么是完美的，它也有无法满足需求的那天，不过我们可以通过自己的方式去改变它。从最早的正则表达式进行修改到后来直接调用 Pandoc API(Application programming interface)，就像一个探索的过程，感觉解决问题后总有莫名的成就感，哈哈。

