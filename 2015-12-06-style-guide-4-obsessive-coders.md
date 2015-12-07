---
layout: post
title: Style Guide for Obsessive Coders
excerpt: "What should be my default coding style?"
modified: 2015-12-06
tags: [coding, programming]
comments: true
---

{% include _toc.html %}


[^1]: <https://en.wikipedia.org/wiki/The_Elements_of_Programming_Style>



对于代码风格的严肃讨论由上世纪70年代《The Elements of Programming Style》[^1]一书开启，它指出风格良好的代码应不仅仅遵循编译通过或个人习惯这样简单的要求，更应当能够被其他人类所理解。也就是说，在代码评判方面，可读性是非常重要的指标。

时光荏苒，对此话题上论述已经无处不在，从纸质出版物到在线课程，从网上广为流传的Google Style Guide到公司SharePoint上的某个Checklist，本文的目的就是将这些建议和规则汇总起来，以进一步发展出自己的代码风格。/* 嗯，处女座的强迫症风格，妥妥的 */



# Readability

风格良好的代码应当易于理解，即有**可读性(readability)**，而**理解代码所花费的时间**则是度量可读性最为重要的标准：

:bulb: **Key Point**    
**可读性基本定理**: 代码的写法应当使别人理解它所需要的时间最小化。    
**The Fundamental Theorem of Readability**: Code should be written to minimize the time it would take for someone else to understand it.
{: .notice}

这儿的**理解**是指能够改动代码、找出缺陷并明白它是如何与其他部分代码进行交互。而且我们应当避开以下几个误区：

* 代码并不总是越短越好，一味的减少代码行数可能让其理解起来花费更久。例如，合并长表达式、删去必要的注释。
* 代码的易读性与效率、架构、可测试性并没有冲突。



# Naming

代码中的每一个名字都应**当作一条小小的注释(think of a name as a tiny comment)**，起一个好的名字可以充分利用这狭小的空间。

:bulb: **Key Point**    
把信息装入名字。    
Pack information into your names.    
{: .notice}

起一个好的名字应当注意以下几个方面：


## Use Specific Words

要选择专业而不是空洞的词。例如：

* `def GetPage(url):` - get信息量不足，无法确定是缓存、数据库还是互联网，考虑用FetchPage或DownloadPage。
* `class BinaryTree { int Size(); };` - size不够明确，没有指明是树的高度、节点数还是占用的内存空间，改进为Height，NumNodes或MemoryBytes。
* `class Thread { void Stop(); };` - stop做的事情没有澄清，对于无法恢复的操作应使用Kill，若有对应的Resume，可以叫Pause。

常用词替换表：

| Word  | Alternatives                                       |
|:-----:|:---------------------------------------------------|
| send  | deliver, dispatch, announce, distribute, route     |
| find  | search, extract, locate, recover                   |
| start | launch, create, begin, open                        |
| make  | create, set up, build, generate, compose, add, new |
{: rules="groups"}


## Avoid Generic Names

tmp、retval和foo、bar之流往往就是想不出名字的托辞，与之相反，我们选用的名字应当能够**描述变量的目的或者它所承载的值**。

* retval包含信息不足，应当使用描述该变量的值的名字来替代。例如，用sum_squares来表示平方和。
* tmp只应用于短期存在并且临时性为其主要存在因素的变量。例如，交互两个变量的值，其他情况下user_info和tmp_file这样的变量名则更佳。
* 与用i、j、k、it和iter作索引/循环迭代器相比，使用更精确的名字可以让代码中的缺陷更为明显。例如，用clubs_i、members_i、users_i或ci、mi、ui来替代i、j、k。
* 如果理由恰当，这样空泛的变量名并非不能使用。


## Use Concrete Names


## Attach Important Details 


## Use Longer Names for Larger Scopes


## Use Capitalization, Underscores to Convey Meanings


## Use Names That Can't Be Misconstrued

