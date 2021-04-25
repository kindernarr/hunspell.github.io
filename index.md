# 向德语系学生介绍hunspell

本文准备长期更新。

我们学习hunspell的目的，主要是为了研究词典软件里面的词形还原，而不是为了拼写检查。

我将需要用到的文件全部放在以下链接：

[hunspell_in_depth](https://pan.baidu.com/s/1HDlBubR1IIGb14v0i_p3Dg) 提取码: rc9v

在这些文件中，manual文件夹的**man 4 hunspell.pdf**最重要，其次重要的是tests.rar中的示例。



首先明确两点：

1. hunspell是不考虑上下文的单词拼写检查工具。
2. hunspell是基于规则而非概率的词形还原工具。

语料处理中用到几种工具，在此区分一下：

1. tokenizer

   tokenizer把一句话中用空格或标点隔开的词一个个断开，比如：

   Wir sind nicht Staub im Wind. 

   Wir, sind, nicht, Staub, im, Wind.

2. lemmatizer	把词语的屈折形式转化为原形，比如：

   findet -> finden.

3. stemmer	把一个词的词干提取出来，本文中不作“词根、词干、词基”等区分，把一个词去掉词尾的部分称为词干，比如： finden -> find.

4. compound word breaker	把复合词中的组成成分剥离出来，比如德语中“引号”是Anführungszeichen，那么： Anführungszeichen -> Anführung, s, Zeichen. 其中，s是Fugenelement 。
   
5. part-of-speech tagger	词形标注工具，能把一个词是阴性还是阳性，是单数第三人称现在时，还是负数第一人称过去完成时等语言信息分析出来，此类工具的代表是[TreeTagger](https://www.cis.lmu.de/~schmid/tools/TreeTagger/). Hunspell只能实现非常简陋的语法标注能力，而且目前我们实际使用的德语词典文件和规则文件都不提供语法信息。

6. grammar checker	语法检查工具。比如office的德语语法检查插件能够以最新正字法为标准，对一篇文章进行语法检查。



tests文件夹里有许多子文件夹，每个子文件夹提供一个实例，往往涉及到man 4 hunspell中提到的一方面。
一个子文件夹内的文件的文件名有着相同的开头，但扩展名不同。
比如：

- needaffix2.dic
- needaffix2.aff
- needaffix2.morph
- needaffix2.good



我们先看看这些扩展名：

- .dic	词典文件，里面的内容类似纸质词典里的词头(lexem)。
- .aff	还原规则集，里面写了怎样把一个加了前缀后缀或与其他词语复合的复杂形式，转化为一个词典中存在的词头的许多组规则。
- .morph	Hunspell程序在Linux Shell里面运行的结果。
- .good	能通过语法检查的正确词形。
- .wrong	不能通过语法检查的错误词形。
- .sug	针对错误词形或正确但罕见的词形，hunspell会认为用户拼错了，会推荐一些它认为正确的词形供采纳。

# 关于option和flag
打开一个普通的.aff文件，我们能发现一些整个词大写的命令，它们位于一行的开头，比如：
```
# 以下内容节选自from_mdict_utf8/de_DE_frami.aff
PFX i Y 1
PFX i 0 -/coyf .

SFX j Y 3
SFX j 0 0/xoc .
SFX j 0 -/zocf .
SFX j 0 -/cz .
```
其中，PFX和SFX都被称为options（本教程中一般会用与**man 4 hunspell**1一致的形式，当我觉得官方的说法很容易引发歧义，或者在一个重要概念上缺乏说法时，我会用自创的术语），PFX是prefix的缩写，代表一套前缀规则；SFX是suffix的缩写，代表一套后缀规则。在**man 4 hunspell**（hunspell讲规则集和词典文件格式的帮助文档）中，有时会用AF作为PFX和SFX的总称，AF是affix的缩写，*affix*包括prefix和suffix，你在.aff或.dic文件中，是找不到AF这个option的。
`PFX i Y 1`中的i，是这一套前缀规则的识别码，i被称为flag，可以看作这套前缀规则的名字。如果你使用过windows或linux的命令行，应该很容易明白flag的含义。其实，这里的flag在一套标准（当我说“一套标准”时，指的是规则集加上词典，也就是.aff和.dic文件）中，具有唯一性，因此你可以将其类比为html中的id。

因此，option是类别，对应hunspell中的某个功能；flag是具体某个规则的识别码。

（为了保证术语与**man 4 hunspell**一致，今后文中我就不翻译option和flag这两个词了）

我们注意到，上面的文件可以分成两个规则“块”，第一格规则块的头部是`PFX i Y 1`，下面的内容只有一条；第二个规则块的头部是`SFX j Y 3`，下属的内容有三条。我们作以下区分：

`PFX i Y 1`这种，称为affix class header。

   ```
   PFX i Y 1
   PFX i 0 -/coyf .
   ```

   以上构成一个代码块。而PFX是这个代码块的option，i是这个代码块的flag。

   除PFX block和SFX block，还有REP或BREAK这两种option的代码块，我不妨称之为REP block和BREAK block：

   ```
   # 以下内容节选自from_mdict_utf8/de_DE_frami.aff
   # 以下是一个REP block，有28条规则隶属于这个代码块。
   REP 28
   REP f ph
   REP ph f
   REP ß ss
   REP ss ß
   REP s ss
   REP ss s
   REP i ie
   REP ie i
   REP ee e
   REP o oh
   REP oh o
   REP a ah
   REP ah a
   REP e eh
   REP eh e
   REP ae ä
   REP oe ö
   REP ue ü
   REP Ae Ä
   REP Oe Ö
   REP Ue Ü
   REP d t
   REP t d
   REP th t
   REP t th
   REP r rh
   REP ch k
   REP k ch
   ```

   ```
   # 以下内容节选自from_mdict_utf8/de_DE_frami.aff
   # 以下是一个BREAK block，有2条规则隶属于这个代码块。
   BREAK 2
   BREAK -
   BREAK .
   ```

   我们注意到，REP block和BREAK block的头部比SFX block简单，仅由option和下属规则的数量组成，没有flag，因为REP block和BREAK block在一个.aff文件中只有一组，不需要名字来区分。

为了避免歧义，今后我们称`BREAK 2`为BREAK block header；称

```
BREAK -
BREAK .
```

为BREAK block body；称其中每一行为BREAK block line。hunspell官方帮助文档中的概念的使用常常较为随意，新手有时候不理解哪些是同义词，哪些又有区别，本教程试图消除这种术语上的混乱。




flag一般用在两个地方：

1. flag用在.dic文件中，比如在from_mdict_utf8/de_DE_frami.aff中，有`Ähren/hij`这个词，也就是说，我们要对Ähren这个词使用h, i, j这三个规则。这三个规则，可以来自任何一种option，可以是SFX。许多人一开始可能会对如此紧凑的写法感到疑惑，觉得这样写太不便于理解和维护了，hunspell的文件格式确实存在这个问题，因为hunspell讲究的是代码执行的效率，为此不得不牺牲一些可读性。

   这种flag用在.dic文件中的用法，在**man 4 hunspell**中称为word with flag。

2. flag用在.aff文件中SFX或PFX规则的第四个位置上。

   所谓第四个位置，是指假如以空格符为分隔符，将SFX或PFX规则的一行断开，比如：

   `SFX j 0 0/xoc .`

   断开后成为：

   `SFX`  `j` ` 0` `0/xoc` `.`

   变成5个字符串。

   `0/xoc`就是所谓的“SFX或PFX规则的第四个位置”，此处，链接到x, o, c这三个option

   这种flag用在.aff文件的SFX或PFX规则中的



下面我们来看真正的hunspell的词典中有哪些options：


| aff_file        | SFX  | PFX  | REP  | ICONV | MAP  | BREAK | SET  | TRY  | NOSUGGEST | WORDCHARS | COMPOUNDRULE | ONLYINCOMPOUND | COMPOUNDMIN | FORBIDDENWORD | NEEDAFFIX | KEEPCASE | CIRCUMFIX | CHECKSHARPS | COMPOUNDBEGIN | COMPOUNDMIDDLE | COMPOUNDEND | COMPOUNDPERMITFLAG | FLAG | OCONV | KEY  | FULLSTRIP |
| --------------- | ---- | ---- | ---- | ----- | ---- | ----- | ---- | ---- | --------- | --------- | ------------ | -------------- | ----------- | ------------- | --------- | -------- | --------- | ----------- | ------------- | -------------- | ----------- | ------------------ | ---- | ----- | ---- | :-------- |
| de_AT_frami.aff | 437  | 68   | 29   | 0     | 0    | 3     | 1    | 1    | 1         | 1         | 0            | 1              | 1           | 1             | 1         | 1        | 1         | 1           | 1             | 1              | 1           | 1                  | 0    | 0     | 0    | 0         |
| de_CH_frami.aff | 437  | 68   | 29   | 0     | 0    | 3     | 1    | 1    | 1         | 1         | 0            | 1              | 1           | 1             | 1         | 1        | 1         | 1           | 1             | 1              | 1           | 1                  | 0    | 0     | 0    | 0         |
| de_DE_frami.aff | 437  | 68   | 29   | 0     | 0    | 3     | 1    | 1    | 1         | 1         | 0            | 1              | 1           | 1             | 1         | 1        | 1         | 1           | 1             | 1              | 1           | 1                  | 0    | 0     | 0    | 0         |
| en_GB.aff       | 1238 | 36   | 28   | 2     | 0    | 0     | 1    | 1    | 1         | 1         | 3            | 1              | 1           | 0             | 0         | 0        | 0         | 0           | 0             | 0              | 0           | 0                  | 0    | 0     | 0    | 0         |
| en_US.aff       | 59   | 14   | 91   | 2     | 0    | 0     | 1    | 1    | 1         | 1         | 3            | 1              | 1           | 0             | 0         | 0        | 0         | 0           | 0             | 0              | 0           | 0                  | 0    | 0     | 0    | 0         |
| es_ES.aff       | 6675 | 80   | 21   | 0     | 6    | 0     | 1    | 1    | 0         | 0         | 0            | 0              | 0           | 0             | 0         | 0        | 0         | 0           | 0             | 0              | 0           | 0                  | 1    | 0     | 0    | 0         |
| fr.aff          | 5384 | 215  | 111  | 43    | 26   | 8     | 1    | 1    | 1         | 1         | 0            | 0              | 0           | 1             | 1         | 1        | 1         | 0           | 0             | 0              | 0           | 0                  | 1    | 2     | 1    | 1         |
| ru_RU.aff       | 1606 | 0    | 0    | 0     | 0    | 0     | 1    | 1    | 0         | 0         | 0            | 0              | 0           | 0             | 0         | 0        | 0         | 0           | 0             | 0              | 0           | 0                  | 0    | 0     | 0    | 0         |



尽管这几种常见的西方语言的hunspell文件中用到的option并不多。

但本教程中会尽量地去讲github上hunspell源代码中[tests](https://github.com/hunspell/hunspell/tree/master/tests)文件夹内示例用到的各种options。
