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

