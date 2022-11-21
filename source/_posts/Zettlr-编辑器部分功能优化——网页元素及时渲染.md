---
title: Zettlr 编辑器部分功能优化——网页元素及时渲染
hide: false
tags:
  - Zettlr
categories:
  - Linux
math: false
mermaid: false
date: 2022-05-12 20:20:04
index_img:
excerpt: 初步实现 Zettlr 的网页标签渲染
---
# 前言

<ruby>Zettlr<rt>/ˈsetlər/</rt></ruby> 服务的是写作者和科研工作者，这类受众通常不会使用花里胡哨的东西，所以 [Zettlr](https://zettlr.com "Zettlr") 减少了很多方面的扩展。举个例子，在 Zettlr 中使用 <ruby>HTML（超文本标记语言）<rt>HyperText Markup Language</rt></ruby>的标签不会得到渲染，你输入 `<button>这是一个按钮</button>` ，它会原样输出。

![原生 Zettlr ](http://101.200.84.36/images/2022/05/12/202205122125084.png "原生 Zettlr ")

写博客的时候时不时会使用到英文简写（一般不大适合使用中文），这时候我会给英文简写做个备注，比如把简写的原文写下。按理来说，原文应该使用括号写出，但是英文原文太长会影响排版，因此我会选择使用 `<ruby>` 标签把原文写在简写和中文的上方，减少页面占用。这种时候 Zettlr 不渲染就会显得比较难看。

因此很早之前我就考虑修改 Zettlr，自己渲染网页元素。可惜，我没接触过 [Electron](https://www.electronjs.org/ "Electron") 开发，不了解这种应用的基本运行框架，而 Zettlr 就是用 Electron 开发的。

前一阵子对 Zettlr 的部分功能进行了优化，感觉自己可以再试试，搞不好可以实现自己很久以来的想法。于是今天下午花了点时候，大概摸清 Zettlr 渲染的部分代码，然后就直接动手，想不到还真的让我这只瞎猫碰到了死耗子！

修改代码之后，现在自己编译的 Zettlr 已经可以渲染网页标签——还可以自定义标签，当然是在<ruby>资源管理<rt>Asset Manager</rt></ruby>中配置自定义的 <ruby>CSS（层叠样式表）<rt>Cascading Style Sheets</rt></ruby>。

![改造后的 Zettlr ](http://101.200.84.36/images/2022/05/12/202205122138784.png "改造后的 Zettlr ")

# 修改

和上次的一样准备 Zettlr 源码，建议直接使用上一次的，免得还得把之前的修改放进来。要修改的文件位置在 Zettlr 源码根目录下的

```bash
source/common/modules/markdown-editor/hooks/render-elements.ts
```

使用你习惯的编辑器打开这个 TypeScripts 文件。

首先在开头从 `can-render-element` 文件引入 `canRenderElement` 函数。

```bash
import canRenderElement from '../plugins/util/can-render-element'
```

![引入需要的函数](http://101.200.84.36/images/2022/05/12/202205122141485.png "引入需要的函数")

接下来在 `renderElements` 函数的最后调用 `renderElems` 函数，函数是我们要写的，所以不用担心。

![调用添加的渲染](http://101.200.84.36/images/2022/05/12/202205122144351.png "调用添加的渲染")

在 `renderElememts` 上方写一个函数，叫做 `renderElems` 。

```bash
// 这是我们的函数
function renderElems (cm: CodeMirror.Editor): void {

  // 获取视图，里面包含编辑器的原始文本
  const viewport = cm.getViewport()
  // 按行读取文本
  for (let i = viewport.from; i < viewport.to; i++) {
  // 无关的内容不用渲染
    if (cm.getModeAt({ 'line': i, 'ch': 0 }).name !== 'markdown-zkn') continue

    const line = cm.getLine(i)
    // 正则表达式获取标签名和内容
    for(const match of line.matchAll(/<\b([^>]+)>(.*?)<\/\1\b>/g)) {

      const curFrom = { 'line': i, 'ch': match.index as number }
      const curTo = { 'line': i, 'ch': match.index as number + match[0].length }
     // 判断是否该渲染。这里需要注意不能落下，因为当你的鼠标靠近标签的时候不该渲染
     if (!canRenderElement(cm, curFrom, { line: curTo.line, ch: curTo.ch + 1 })) {
       continue
     }

      // 通过标签名创造一个标签
      const tag = document.createElement(match[1])
      tag.innerHTML = match[2];

      // 这个用于用于标记标签元素是否该被渲染
      const marker = cm.markText(
        curFrom, curTo, {
          'clearOnEnter': true,
          'replacedWith': tag,
          'inclusiveLeft': false,
          'inclusiveRight': false
         })
      // 因为已经渲染了标签，所以广播编辑器修改元素所占的位置
      marker.changed() // Notify CodeMirror of the potentially updated size
 
      // 如果鼠标点击，或者光标靠近标签，就清除渲染的标签，回归原来的状态
      tag.onclick = (e) => {
        marker.clear()
        cm.setCursor(cm.coordsChar({ left: e.clientX, top: e.clientY }))
        cm.focus()
       }
     }
  }
}

# 下面是原有的 renderElements 函数
function renderElements (cm: CodeMirror.Editor): void {
...
}
```

现在只需要重新编译就可以实现网页标签的渲染了，上一篇博客介绍过怎么编译。

# 后语

修改完 Zettlr 之后，我如愿彻底离开 Typora 了。不过说来，Typora 在 Manjaro 居然有 200M 以上，远比 Zettlr（112M）大。
