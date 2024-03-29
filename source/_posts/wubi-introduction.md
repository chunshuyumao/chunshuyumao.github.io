---
title: 五笔输入法入门
hide: false
tags:
  - 五笔
  - 输入法
categories:
  - 杂谈
math: false
mermaid: false
date: 2023-04-17 19:32:09
---

# 前言

在现在这个智能时代，拼音输入法已经非常快了，而且在某种程度上，拼音输入法更适合普通人的输入 —— 你怎么说就怎么打。然而专业打字或者生僻字输入，又或者小词组、单字输入，五笔的优点还是很明显的。正常情况下我都使用拼音输入法，但是碰到上面的的问题时，我基本是五笔解决。

五笔输入法有一个学习门槛。不过不是五笔难学，而是教五笔的人不知道怎么教。上来就是一堆字根，大家什么都不知道就直接背字根，自然就和背英语单词一样。

我去年曾决定学五笔，在学习过程中还是发现了很多有趣的东西。因此今天打算把五笔学习的一些技巧分享出来，可能并不科学，但我觉得对于五笔入门还是足够的。看了之后多久能够熟练打五笔我不敢说，但是看完一个小时内你能用五笔打字是肯定的。

五笔一般来说有三种：86 五笔、98 五笔和新世纪五笔。其中 86 五笔出现最早，普及比较高，98 五笔重码率较低，新世纪五笔，嗯，对的。我学的是 86 五笔，所以后面的教程只对 86 五笔有用。如果不知道自己用的是什么，一般来说就是 86 五笔。

# 五笔入门

## 五笔的基本知识

开始五笔之前，我们需要回想一下小学的知识，汉字基本笔画有几笔？五笔，分别是横、竖、撇、点、折。请用小学知识，不要想“永字八笔”是哪八笔。

五笔输入法叫五笔，不是需要敲键五次，而是使用了汉字的五个基本笔画。请看下面的图：

![五笔字根图](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/wubi1.jpg)

![五笔字根分区](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/wubi.jpg)

上面这个五笔字根图，将五笔分为五个区域，每个区域有五个按键：

1. 横区，G-A 五个按键，其中 G、F、D、S 和 A 又分别对应汉字五笔：横、竖、撇、点、折。
2. 竖区，H-M 五个按键，其中 H、J、K、L 和 M 又分别对应汉字五笔：横、竖、撇、点、折。
2. 撇区，T-Q 五个按键，其中 T、R、E、W 和 Q 又分别对应汉字五笔：横、竖、撇、点、折。
2. 点区，Y-P 五个按键，其中 Y、U、I、O 和 P 又分别对应汉字五笔：横、竖、撇、点、折。
2. 折区，N-X 五个按键，其中 N、B、V、C 和 X 又分别对应汉字五笔：横、竖、撇、点、折。

||横|竖|撇|点|折|
|:--:|:--:|:--:|:--:|:--:|:--:|
|横区|G|F|D|S|A|
|竖区|H|J|K|L|M|
|撇区|T|R|E|W|Q|
|点区|Y|U|I|O|P|
|折区|N|B|V|C|X|
: 五笔的结构，根据字根的第一第二划就可以找到按键

二十六个字母多一个 Z，用来做万能建。

好的，五笔的知识说完了。

## 拼字规则

知道五笔的规则就行了，我们开始拼字。五笔有几个规则，之前在王码的网站能够下载五笔的教材，人家还总结了几个规则，现在找不到了，我就自己总结规则：

1. 分字根。五笔字根上经常出现的杂乱无章的部首，一般是汉字可以分化的最小单位，比如“木”。大家不用担心怎么区分字根，其实不用，就当作一个字你觉得能分几个就行了。举个例子，“字”能分成几个？多半就是“宀”和“子”，这就是字根了。
1. 写字。用脑子写字就行了，按照字根的第一第二划找按键。要确定某个区域就看字根的第一划，要确定某个键就看第二划。比如“白”是一个字根，第一划是“丿”，在撇区（T-Q），第二划是“丨”，在 TREWQ 横竖撇点折 —— 第二个，也就是 R。好的，你可以看看“白”是不是在这里。再试试，“王”字在哪里？1、2、3、4、5 ，“王”的第一划是“一”，所以在横区（G-A），第二划是“一”，所以对应 GFDSA 横竖撇点折 —— 第一个，也就是 G。看到了吧。“大” 呢？横撇 —— 所以是 D。
1. 不够四键就按照字的结构区分。字有左右、上下和杂合三种结构，分别分布在每个区的第一、二、三键。举个例子，“的”是左右结构，找第一个字根“白”是“撇竖” R，第二个字根“勹”是“撇折” Q，第三个字根是“丶” Y，所以“的”是 RQY。不够四笔，加一个码，叫做“补码”，左右结构，按字的最后一笔所在的区域的第一、二、三键区分结构。“的”最后一笔在点区，还是左右结构，左右、上下、杂合在点区分别对应 Y、U、I，所以就是 Y，最后确定“的”是 RQYY。
1. 同一笔画构成的字根，有多少划，字根就在第几个按键。举个例子，“三”都是由“一”构成，有三划，所以在横区（GFDSA）的第三个，也就是 D，同理“二”有两划在第二个 F，“一”只有一划在第一个 G。再看“丨”在竖区（HJKLM），有一划在 H，“刂”有两划在 J，“Ⅲ” 三划，在 K。有个反例是“目“”日“”口”，中间的笔画依次减少，但却在分别在 H、J、K。
1. 猛的一看，看起来差不多的字在一个区域。这就需要大家的抽象能力了。比如“五”“(青字头)”和“王”差不多，所以都在 G。“川” 和 “Ⅲ”差不多所以在 K，类似的还有“大”“犬”“石”和“古”，都差不多，所以在 D。

## 学以致用

学完了，开始打字，拼写。

#### 布

可以分为(横撇)、冂和丨。

![布 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419111543.png)

1. (横撇)：第一第二划“横撇”在横区（GFDSA）第三个，即 D。
2. 冂：第一第二划“竖勾”在竖区（HJKLM）第五个，即 M。
3. 丨：“竖”在竖区（HJKLM）第一个，即 H。
4. 结构：最后一笔在竖区，是上下结构（(横撇)和巾），对应第二个 J。

“布”的五笔是 DMHJ。

#### 努

可以分为女、又和力。

![努 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419111840.png)

1. 女：第一第二划“折撇”在折区（NBVCX）第三个，即 V。
2. 又：第一第二划“折捺（点）”在折区（NBVCX）第四个，即 C。
3. 力：特殊记忆，“力”在 L 键，毕竟它读作 LI。
4. 结构：上下结构，最后一划是丿但是 86 五笔搞错了，把笔画弄成了最后一笔是折勾，所以最后一笔在折区（NBVCX），上下结构在第二个，即 B。

“努”的五笔是 VCLB。

>
> 这里的“努”字三个码就可以打完了，因为它是常用字，可以三个码打完叫三级简码。
> 同理，还有二级简码和一级简码，分别只需要敲两个码和一个码。
>

从这里可以看到，86 五笔还是有点问题的，特别是笔画上存在毛病。不过问题不大，因为只打前面三个就出现“努”了。

除此之外，竖区域的字根分布不是很规范，竖区一般特殊记忆，因为竖区很多和“口”字有关。

#### 语

可以分为讠、五和口。

![语 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419112321.png)

1. 讠：讠和“言”相似，根据规则五，可以知道“言”的第一第二划是“点横”，在点区（YUIOP）第一个，即 Y。
2. 五：五和王相似，根据规则五，可以知道“王”在 G, 所以五也在 G。
3. 口：目、日、口特殊记忆，在 H、J、K 键，笔画依次减少，也可以说“口”是 K 音（kou），即 K。
4. 结构：左右结构（言和吾）。最后一笔是横，在横区（GFDSA）第一个，即 G。

“语”的五笔是 YGKG。

#### 是

可以分为日、一和止。

![是 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419114109.png)

1. 日：目、日、口特殊记忆，在 H、J、K 键，笔画依次减少，所以是 J。
2. 一：只有一划，在横区（GFDSA）第一个，即 G。
3. 止：第一第二划是竖横，在竖区（HJKLM）第一个，即 H。
4. 结构：上下结构。最后一划捺（点）在点区（YUIOP），上下结构是第二位，所以是 U。

“是”的五笔是 JGHU。“是”属于一级简码，只需要一个 J, 也可以当作三级简码 JGH，或者全码 JGHU。

#### 中

可以分为口、丨。

![中 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419112404.png)

1. 口：目、日、口特殊记忆，在 H、J、K 区，口是 K。
2. 丨：第一划是竖，位于竖区（HJKLM），笔画是一，所以是 H。
3. 结构：杂合结构，最后一划是竖，在竖区（HJKLM）第三个，即 K。

“中”的五笔是 KHK。“中”字属于一级简码，只需要一个 K 即可，也可以当作二级简码，也就是 KH，或者三级简码 KHK。

#### 国

可以分为囗、王和丶。

![国 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419112814.png)

1. 囗：特殊记，在 L，这和“口”不是一个字，是“围”的古字“囗（wei）”，全包围结构使用，“口（kou）”用于偏旁。
2. 王：第一划第二划是横横，位于横区（GFDSA）第一个，即 G。
3. 丶：只有一划是点，位于点区（YUIOP）第一个，即 Y。
4. 结构：杂合结构。最后一笔点，杂合结构在点区（YUIOP）第三个，即 I。

“国”的五笔是 LGYI。“国”字属于一级简码，只需要一个 L 即可，也可以是三级简码 LGY，或者全码 LGYI。

这里又出现了竖区（HJKLM）的特殊字符“囗（wei）”，位于 L，同在这里的还有“力”“田”“甲”“车”“皿”“罒”“四”，这些字都差不多。例如“皿”“罒”和“四”，“田”“甲”和“车”。当然，有人肯定会疑惑，“车”哪像了？相信我，真的像。

#### 少

可以分为小、丿。

![少 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419114943.png)

1. 小：“小”类似“氺”和”水“，”水“类似于”氵“所以三点水由三个点构成在点区（YUIOP），满足规则四，不用找了，我下面附有。所以是 I。
2. 丿：撇区（TREWQ）第一个，所以是 T。
3. 结构：上下结构，最后一划在撇区（TREWQ），上下结构是第二个，即 R。

>
> 4. 同一笔画构成的字根，有多少划，字根就在第几个按键。举个例子，“三”都是由“一”构成，有三划，所以在横区（GFDSA）的第三个，也就是 D，同理“二”有两划在第二个 F，“一”只有一划在第一个 G。再看“丨”在竖区（HJKLM），有一划在 H，“刂”有两划在 J，“Ⅲ” 三划，在 K。有个反例是“目“”日“”口”，中间的笔画依次减少，但却在分别在 H、J、K。
> 

“少”的五笔是 ITR，属于二级简码可以 IT 打出，也可以当作三级简码 ITR 打出。

#### 数

可以分为米、女和攵。

![数 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419115841.png)

1. 米：米有四个点，和“灬(四点底)”有点类似，都是点，还有四划，按照规则四知道在点区（YUIOP）第四个，即 O。类似的字体还有”火“、“亦”字底、“业”字头。
2. 女：第一第二划是折撇，在折区（NBVCX）第三个，即 V。同键盘的还有“巛””九““刀”“彐”和”臼“。"刀"类似于”九“，”彐“类似于”臼“。
3. 攵：第一第二划是撇横，在撇去（TREWQ）第一个，即 T。
4. 结构：左右结构，最后一划是捺（点），在点区（YUIOP），左右结构是第一个，所以是 Y。

”数“的五笔是 OVTY。这是一个三级简码字，可以 OVT 打出，也可以使用全码 OVTY 打出。

#### 民

可以分为巳和七。

![民 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419121412.png)

1. 巳：第一第二划是折横，在折区（NBVCX）第一个，即 N。类似的还有“已”、“眉“字头和”心“”忄“”忝“字底和”羽“。”羽“字在这里可能和日常书写中”羽“字一般都写成”ᴲᴲ(两个反过来的E)“有关。
2. 七：”七“类似于”匚“，所以是横折，在横区（GFDSA）第五个，即 A。类似的还有“弋”“工”“戈”“廿”和“(黄字头)”。从“(黄字头)”又衍生出“艹(草字头)”。
3. 结构：杂合结构，最后一笔是勾（折），在折区（NBVCX），杂合是第三个，即 V。

“民”的五笔是 NAV。二级简码，可以直接 NA 打出，也可以当作三级简码 NAV 打出。

#### 族

可以分为方、(撇横)、(撇横)和大。

![族 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419122921.png)

1. 方：第一第二划是点横，在点区（YUIOP）第一个，即 Y。
2. (撇横)：第一第二笔是撇横，在撇区（TREWQ）第一个，即 T。
3. 同上。
4. 大：第一第二笔是横撇，在横区（GFDSA）底三个，即 D。

“族”的五笔是 YTTD。三级简码，可以直接 YTT 打出，也可打全码 YTTD。

### 需要注意的东东

#### 字根很多

我们上面拼字很幸运，没有碰到难字，不过这还是难以避免的，比如“繁”“藏”“疆”“癰”“龘(三个龙的繁体)”。

五笔拼字的更多规则：

1. 字根数多于四个，只选择第一、二、三和最后一个字根。比如“繁”，按照正常拼写是 T（撇横）X（母，没有两点与横）G（一）U（冫，两点）T（攵）X（幺）I（小）。直接选取前三码和最后一码，即 TXGI。

  ![繁 字字根](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419155042.png)

1. 打词组时，二字词组各打前两个码。例如“放假”是 YTWN（方攵亻己）。三字词组前两二字打一码，后一个字打两码。比如“天然港”是 GQIA（一夕氵廿）。四字词组每个字各打一个码，比如“刚正不阿”，直接是 MGGB（冂一一阝）。多字词组，即四字以上，则打前三个字的第一个码和最后一个字的第一个码。如“中华人民民主共和国”是 KWWL（口亻人囗），“美利坚合众国”是 UTJL（丷禾刂囗）。注意，打词组不能使用一级简码。

  ![刚正不阿](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419162102.png)

1. 打字第一定律：能少打一个码就少打一个码。比如“是”，J 可以做到就尽量不选择 JGHU。使用空格键上屏。一级简码字在文章开始第二张图每个字母旁。一共二十五个，不用记，自己试试只打一个键，出来的就是一级简码，用久了自然就记得。

1. 每个区域的笔画名称可以通过区域首个键双击和 LL 打出。比如横（一）是 GGLL，竖（丨）是 HHLL，撇（丿）是 TTLL，点（丶）是 YYLL，折（乛）是 NNLL。

1. 键名。键名指的是按键上可以独立成字的字，位于字根表的左上角。例如“木”在 S 键怎么打出？敲打四次 S 即可，即 SSSS。再如“大”在 D，所以“大”是 DDDD。

1. 偏旁。偏旁是指汉字的组成部分，不是字根，只是和字根很像。比如“幺（绞丝旁）”在 X 键怎么打出？简单，先敲一次偏旁所在的键，然后按照笔画一笔一笔拼出。笔画超过四码就取最后一码做第四码。
  比如“丁”在 S 键，要打出“丁”需要先打 S，然后是横竖（一丨），所以是 SGH。
  再如“耳”在 B 键，先打 B 键，然后是横竖竖横横横（一丨丨一一一），所以取前两码加最后一码，即 BGHG。
  再如“攵”在 T 键，先打 T，然后是撇横撇捺（点）（丿一丿乀），选前两码加最后一码，即 TTGY。

#### 练五笔的技巧

1. 五笔是不是得会写字？

刚学五笔会觉得很无力，因为自己都不会写字了还怎么想字怎么写？其实五笔就是一个习惯的问题。我现在打五笔，一般不会想这个字怎么写，而是习惯就打出来。如果有人问我“不”字怎么打，我可能会卡顿，但是给我一个键盘我就会敲 —— 事实如此。

刚学五笔的时候也碰到过打字太慢的问题，所以练五笔的那一阵子我都没有发博客，因为速度太慢了，我没那么多时间，但是久了习惯了速度自然就上来了。

2. 打词组还是单字？

这个看个人。我刚开始也觉得先练单字，习惯了再换二字、三字……但是我发现自己没有习惯的那天，而是某次誊抄词条发现，自己打词组速度似乎更快。

3. 是不是看了这些技巧就不用记字根了？

不然，练五笔最终还是会回到记词根那一步。但是，但是，学会五笔才知道字根为什么这样，而不是背完字根就知道五笔了。等到你习惯五笔了，你就会知道，原来字根是拿来总结的，不是拿来初学的。即使是现在我也不能说出各个词根在哪里，至少说不全。所谓“王旁青头兼五一，夏芒芒夏暑相连，秋处露秋寒霜降……”

4. 怎么学五笔？

学五笔需要一个字根表，一个网站用于查询五笔。为什么？因为五笔，无论是 86 五笔还是 98 五笔、新世纪五笔，总有一些设计上的不足，顾虑不到汉字的书写顺序。所以你即使拼对了但就是打不出来。五笔，复杂字很轻松就可以打出来，但是简单的字一般就比较难了，所以你需要一个可以查询五笔码的[网站](https://www.52wubi.com "五笔码查询")。

此外，要善用 Z 键。我说过这是一个万能键，实际上是学习键。在大多数输入法中，对于一个你不会拼的字，你可以先输入 Z，然后打拼音，输入法就会提示你这个字的五笔

![Z 键学习键](https://cdn.jsdelivr.net/gh/chunshuyumao/202203@master/2023042/20230419193537.png)

举个例子，“龘（三个繁体龙）”只需要 UEGD（立月一三），“闘”大家多半不会读，但是只需 UGKF（门一口寸）即可，谁在乎它读作什么？“卐（万字符）”只需 NGHG（乙一丨一）。

但简单的字五笔就有点不得行了：戌、戍、戊、戎。我知道有些人连这些字都分不清：横戌(xǖ)点戍(shù)戊(wù)中空，十字交叉读作戎(róng)。

正确拼法是：

1. 戌，DGN 丆一乀
2. 戍，DYNT 丆丶乀丿
3. 戊，DNY 丆乀丶
4. 戎，ADE 弋（横撇），杂合结构

所以多练吧。马克思保佑你！

# 后语

网上的五笔字根图都是有水印的，自己一直没有做五笔教程就是因为使用别人的图需要带水印。前一阵子想自己做一个字根图，但是上面的字根很多都是打不出来的。我也想知道网上的字根图怎么做的，但是没办法，找不到制作方法。

后来索性，一不做二不休，通过遮挡的方式制作字根，也就是选好一个字，然后使用同背景色的东西遮住，勉强做成字根。但是太费时间了，我没有那么多功夫，所以直接作罢，随便到网上选了一张没有水印的，想找一张没有水印的图确实很难。
