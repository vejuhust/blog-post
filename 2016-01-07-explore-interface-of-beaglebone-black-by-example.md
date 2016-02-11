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

本文作为第一篇主要介绍基本概念，并作为后续文章的目录。



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

如上图所示，BBB提供了以太网、USB等多种接口，我们所关注的是标注为**7**的扩展接口(Expansion Headers)。左边称为**P9**，右边为**P8**，各有46个插脚口(pin)，编号从左到右从上到下。提供8种模式(mode)，支持以下通讯协议：

* GPIO
* ADC
* UART
* I2C
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

* GPIO的插图
* 几种GPIO名称的读法



# :notebook: Table of Contents



# :floppy_disk: Resource 


