---
layout: post
title: How Does My blink(1) mk2 Blink?
excerpt: "Notes from a new-comer of ThingM blink(1) mk2"
modified: 2016-01-02
tags: [coding, programming, blink, hardware]
comments: true
---

{% include _toc.html %}


[^1]: ThingM <http://blink1.thingm.com/>
[^2]: ThingM <http://blink1.thingm.com/getting-started/>
[^3]: IFTTT <https://ifttt.com/blink1>
[^4]: IFTTT <https://ifttt.com/channels>
[^5]: Kickstarter <https://www.kickstarter.com/projects/thingm/blink1-mk2-the-usb-rgb-led-improved>
[^6]: ThingM <http://buy.thingm.com/blink1>
[^7]: HPCAC <http://www.hpcadvisorycouncil.com/events/2013/ISC13-Student-Cluster-Competition/>


blink(1) mk2是ThingM[^1]出品的一款可编程的USB小彩灯，主要的用法就是通过接口[^2]或IFTTT[^3]控制它的颜色和亮度。

初识blink(1)是2013年底在IFTTT[^4]上，被这个有着让强迫症患者不安的名字的奇怪channel所种草。无奈当时Kickstarter[^5]上的众筹已过，官网[^6]也显示缺货，只好默默长草。直到最近女友去美国出差为我带回了两套，心中草原才得以拔除。

初遇时的想法是用在集群上来识别机器。2013年初和小伙伴做HPC竞赛[^7]的时候经常遇到这样的困惑——眼前这台服务器到底是集群中的哪个node，或者眼前这堆服务器中的哪一台是我们要找的nodeX。给机器贴标签可能是一个解决方案，但我们会频繁的将不同机器硬盘交换，因此会更加混乱。如果可以给每台机器插都插上blink(1)，给每个node分配一种独有的颜色或发光模式，查找nodeX时直接点亮它的blink(1)，这两个困惑便可轻易解决。/* 当然，导师可能不同意，因为这$30一个的小东东很难开发票 */

时过境迁，我也不再需要亲手运维集群了。这篇文章就来记录一下现在的我对blink(1)的探索。

