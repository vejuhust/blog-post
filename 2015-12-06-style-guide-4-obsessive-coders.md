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
[^2]: <https://google.github.io/styleguide/cppguide.html#Constant_Names>
[^3]: <https://google.github.io/styleguide/cppguide.html#Macro_Names>


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

* retval包含信息不足，应当使用描述该变量的值的名字来替代。例如，用sum\_squares来表示平方和。
* tmp只应用于短期存在并且临时性为其主要存在因素的变量。例如，交互两个变量的值，其他情况下user\_info和tmp\_file这样的变量名则更佳。
* 与用i、j、k、it和iter作索引/循环迭代器相比，使用更精确的名字可以让代码中的缺陷更为明显。例如，用clubs\_i、members\_i、users\_i或ci、mi、ui来替代i、j、k。
* 如果理由恰当，这样空泛的变量名并非不能使用。


## Use Concrete Names

在给变量、函数或元素命名的时候，应当把它描述的更具体而不是更抽象。例如：

* 对于检测服务是否可以监听某个端口的方法，叫`CanListenOnPort()`比`ServerCanStart()`要好。
* 对于禁止拷贝构造函数或赋值操作符的宏，叫`DISALLOW_COPY_AND_ASSIGN()`比`DISALLOW_EVIL_CONSTRUCTORS()`更妥当。
* 对于输出额外调试信息的命令行参数，叫`--extra_logging`就比`--run_locally`要清晰，如果要建立和使用本地数据库，那么可以另加一个`--use_local_database`。


## Attach Important Details 

变量名常被看作是必读的短小注释，因此可以将必须让读者知道的重要信息添加进去。两个方向：

* 为度量添加度量单位，例如用start\_ms和elapsed\_ms代替start和elapsed来度量时间。
* 添加重要属性的说明，特别是安全缺陷或让人感到意外的属性，例如untrustedUrl或unsafeMessageBody。

度量单位举例：

| Function                      | Variable Name | Better Name  |
|:------------------------------|:--------------|:-------------|
| Start(int delay)              | delay         | delay\_secs  |
| CreateCache(int size)         | size          | size\_mb     |
| ThrottleDownload(float limit) | limit         | max\_kbps    |
| Rotate(float angle)           | angle         | degrees\_cw  |
| Commit(string id)             | id            | hex\_id      |
{: rules="groups"}


附加重要属性举例：

| Situation                              | Variable Name | Better Name         |
|:---------------------------------------|:--------------|:--------------------|
| 纯文本格式的密码，进一步处理前需要加密 | password      | plaintext\_password |
| 用户发布的注释，展示之前需要转义       | comment       | unescaped\_comment  |
| 已转为UTF-8格式的HTML字节              | html          | html\_utf8          |
| 以URL编码的输入数据                    | data          | data\_urlenc        |
{: rules="groups"}


## Use Longer Names for Larger Scopes

* 对于几屏之间都可见的变量不应使用令人费解的单个字母，应相对偏长。
* 对于只在短短几行小作用域里的变量则应使用短名字。
* 变量名应避免使用项目特有的缩略词/缩写，要以团队新成员是否能够理解含义为准则。
* 应删去名字中拿掉而不会有信息损失的冗余单词，例如`ConvertToString()`应改为`ToString()`。


## Use Capitalization, Underscores to Convey Meanings

对不同的实体使用不同的格式就像语法高亮的显示形式一样，能够让你更容易地阅读代码。例如：

* 《Google C++格式规范》要求：
  - kConstantName表示常量，区别于MACRO\_NAME表示的宏[^2]。
  - 类成员变量与普通变量的区别是以下划线`_`结尾[^3]。
* 《JavaScript: The Good Part》指出，构造函数应当首字母大写，普通函数首字母小写。
* jQuery返回的对象也同样加上`$`作为前缀以示区分。
* 在HTML/CSS中，用下划线`_`来分割ID中的单词，class中用连字符`-`。


## Use Names That Can't Be Misconstrued

:bulb: **Key Point**    
仔细审视名字并多问自己几遍：“这个名字会被别人解读成其他含义吗？”    
Actively scrutinize your names by asking yourself, "What other meanings could someone interpret from this name?"
{: .notice}

我们应当小心处理可能会有歧义的名字。例如：

* 避免用filter: 因为它的含义可能是“挑出”或者是“减掉”。
* 用truncate代替clip: 因为clip的含义可能是“从尾部删除length的长度”或“截取最大为length的一段”。
* 给要限制的事物加上min\_或max\_前缀表示包含在内的极限，limit有“少于”和“少于等于”的二义性。
* 对于前后都被包含的范围(e.g. `[1, 4]`)，用first/last来表示，或者考虑min/max。
* 对于包含/排除的范围(e.g. `[1, 5)`)，用begin/end来表示。
* 为布尔值命名，使用is、has、can或should一类的词，避免使用否定意义的词，例如用use\_ssl而非disable\_ssl。
* 用户期望get()和size()都是轻量级的方法，如果是O(n)的时间复杂度要叫做compute()和count()。

