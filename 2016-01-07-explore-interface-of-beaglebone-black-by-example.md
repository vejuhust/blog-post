---
layout: post
title: Explore Interface of BeagleBone Black by Example
excerpt: "My incomplete programming guide to play with hardware on BBB"
modified: 2016-01-07
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}


与[Raspberry Pi](https://www.raspberrypi.org/)和[Arduino](https://www.arduino.cc/)相比，[BeagleBone Black](http://beagleboard.org/black) (a.k.a. BBB)的一大优势就是扩展接口相对较多。这些PC或Mac所不具备的扩展接口可以让你与各类嵌入式设备交互，更加接近物理世界，获得与纯软件编程所不同的体验和乐趣。

本系列文章就以多个小实验的实现来探讨如何通过BBB来操作各类设备的。但这并不是一套完整的教程，而更像是独立探索时写下的笔记。

本文作为第一篇主要介绍基本概念，并作为后续文章的目录。关于新入手的BBB如何安装、设置请参考[Start Guide for BeagleBone Black](http://yewei.io/)一文。



# :bookmark: Tips 

作为一名软件工程师，在这几周探索的过程中如果说有什么经验教训的话，或许就是以下这几条了：

1. 小心静电，断电连线。
2. 仔细阅读正确的datasheet。
3. 通电之前检查连线，检查，检查，再检查！

重要的事情说三遍，第三条是非常重要的。在软件开发领域，大多数时候运行程序之前我们并不会刻意检查正确性，因为试错的成本非常之低，重新再运行一遍也不过几秒钟到几分钟的事情。而在硬件领域则不大相同，错误的连线可能会烧毁设备，带来时间和金钱上可观的损失。



# :dog: Headers

<figure>
  <a href="/images/photo/beaglebone/bbb-front-back.jpg">
    <img src="/images/photo/beaglebone/bbb-front-back.jpg" alt="Front & Back of BeagleBone Black">
  </a>
</figure>

如上图所示，BBB提供了以太网、USB等多种接口，我们所关注的是标注为7的扩展接口(Expansion Headers)。左边一排被称为**P9**，右边为**P8**，各有46个端口(pin脚)，编号从左到右从上到下。提供8种工作模式(mode)，支持以下通讯协议：

* GPIO
* ADC
* UART
* I²C
* SPI

电气参数方面：

* GPIO口的电压为3.3V
  - 最大输入大约为8mA
  - 最大输出为4-6mA
* VDD口的输出电压为3.3V或5V
  - 3.3V插脚口最大输出电流为250mA
  - 5V插脚口最大输出电流为1A
  - 5V插脚口仅在电源适配器供电的情况下可以使用
* ADC口的电压为1.8V



# :on: GPIO

General-Purpose Input/Output (a.k.a. GPIO)顾名思义就是通用型的输入输出端口，可以提供基本数字信号的输入和输出。它是BBB扩展接口中各个pin脚8种工作模式的一种，下图列出了各个pin脚默认的工作模式(不同操作系统之间可能有差异)，更详细的内容可以参考[Derek Molloy制作的表格](https://github.com/vejuhust/beagle-code/tree/master/datasheet/beaglebone-black)。

对于BBB上的具体pin脚，常见的有四种方式指代，例如：

| Header Pin | GPIO Name | Signal Name | Kernal Pin |
|:----------:|:---------:|:-----------:|:----------:|
| P8\_12     | GPIO1\_12 | 44          | 12         |
| P9\_11     | GPIO0\_30 | 30          | 28         |
| P9\_27     | GPIO3\_19 | 115         | 105        |
| USR0       | GPIO1\_21 | 53          | 21         |
{: rules="groups"}

具体含义如下：

* Header Pin:
  - P8_12表示右边P8上的12号pin脚
  - USR0表示以太网口右边四枚LED中最右边的那枚
* GPIO Name: GPIO0\_30表示AM335x上0号GPIO控制芯片中偏移量为30的pin脚
* Signal Name: 用于Linux，可以由GPIO Name计算而得
  - 以GPIO1\_12为例，算得: 1*32+12=44
  - 以Bash为例，可直接使用: `echo 66 > /sys/class/gpio/export`
* Kernal Pin: Linux内核中表示pin脚的方法，常用于Device Tree相关
  - 通过`cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins`可以查看所有状态

<figure>
  <a href="/images/photo/beaglebone/bbb-pin-default.png">
    <img src="/images/photo/beaglebone/bbb-pin-default.png" alt="Default Pin Configuration of BeagleBone Black">
  </a>
</figure>



# :notebook: Table of Contents

* [LED (Light-Emitting Diode)](/explore-interface-of-beaglebone-black-w-led/)
  - [Blink On-Board LED](/explore-interface-of-beaglebone-black-w-led/#blink-on-board-led)
  - [Blink External LED](/explore-interface-of-beaglebone-black-w-led/#blink-external-led)
  - [Dim the LED](/explore-interface-of-beaglebone-black-w-led/#dim-the-led)
* [Button](/explore-interface-of-beaglebone-black-w-button/)



# :floppy_disk: Resource 

* [BeagleBoard.org](http://beagleboard.org/black): 官方网站，有各类资源的索引
* [eLinux.org](http://www.elinux.org/Beagleboard:BeagleBoneBlack): 官方wiki页面，有技术资料的索引
* [Texas Instruments](http://www.ti.com/tool/BEAGLEBK): TI的产品介绍页面，有购买方式和文档的索引
* [CircuitCo@GitHub](https://github.com/CircuitCo/BeagleBone-Black/): 制造商开源的BBB及扩展板的硬件资料
* [Adafruit](https://learn.adafruit.com/category/beaglebone): 良心企业，不仅仅是卖板子，还是布道师
* [Adafruit@GitHub](https://github.com/adafruit/): Adafruit开发的各类库的源代码
* [Fritzing](http://fritzing.org/): 开源的电子设计工具，从面包板到PCB都可以画
* [Getting Started with BeagleBone](http://shop.oreilly.com/product/0636920028116.do): Matt Richardson写的入门教程，以Python为主，轻松简单，GPIO有涉及但不深入
* [Exploring BeagleBone](http://as.wiley.com/WileyCDA/WileyTitle/productCd-1118935128,subjectCd-CM43.html): Derek Molloy写的全面教程，基于C++，内容深入浅出，还有[配套网站](http://exploringbeaglebone.com/)
