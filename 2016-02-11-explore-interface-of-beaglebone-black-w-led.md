---
layout: post
title: Explore Interface of BeagleBone Black with LED
excerpt: "It's all about LED: blink, turn on/off, dim"
modified: 2016-02-11
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}


Light-Emitting Diode (a.k.a. LED)中文叫作发光二极管，是一种能够发光的半导体电子元件，在近些年被普遍用作照明用途之前，曾主要用作指示灯及显示板。这次我们使用LED目的就是来指示电路是否正确的接通，堪称“Hello World”实验。



# Blink On-Board LED

在使用扩展接口连接外部LED之前，我们先来探索一下BBB板载的LED，以确认开发环境正常。


## On-Board LED

<figure>
  <a href="/images/photo/beaglebone/bbb-onboard-led.png">
    <img src="/images/photo/beaglebone/bbb-onboard-led.png" alt="On-Board LED of BeagleBone Black">
  </a>
</figure>

如上图所示，BBB板载有5枚蓝色LED，以太网口左边的是PWR，右边USR3到USR0依次排开。他们默认的作用如下：

* PWR: BBB启动成功后会常亮；如果启动失败，BBB会在其闪烁后关机
* USR0: 展示来自Linux内核的心跳包
* USR1: 闪烁表示microSD卡是否正在被访问
* USR2: 闪烁表示Linux内核没有处于idle状态
* USR3: 闪烁表示板载eMMC是否正在被访问


## Environment

接好BBB通上电，确保网络可以连通。待BBB启动完成后，通过其他设备的浏览器直接访问其80端口，稍等片刻，可以看见如下的介绍页面：

<figure>
  <a href="/images/photo/beaglebone/webportal-screen.png">
    <img src="/images/photo/beaglebone/webportal-screen.png" alt="Screenshot of Web Portal on BBB">
  </a>
</figure>

页面上内容是一个BoneScript入门的教程，可以直接跳到“Cloud9 IDE”部分，按提示在Cloud9中运行样例代码，如下图所示。

<figure>
  <a href="/images/photo/beaglebone/cloud9-screen.png">
    <img src="/images/photo/beaglebone/cloud9-screen.png" alt="Screenshot of Cloud9 IDE on BBB">
  </a>
</figure>

由于整个环境启动较慢，点击Run之后请耐心半分钟，可以看见USR0、USR1和USR2灯熄灭，USR3灯在规律闪烁。若觉得单个LED闪烁可能不够明显，请参考[以下代码](https://github.com/vejuhust/beagle-code/blob/master/experiment/blink-led/myblink.js)将其修改成四个User LED同闪烁。至此，基本确认开发环境可用。

{% highlight javascript %}
var b = require('bonescript');

var leds = ["USR0", "USR1", "USR2", "USR3" ];
var state = b.LOW;

for (var i in leds) {
    b.pinMode(leds[i], b.OUTPUT);
}

setInterval(toggle, 500);

function toggle() {
    if (state == b.LOW) {
        state = b.HIGH;
    }
    else {
        state = b.LOW;
    }

    for (var i in leds) {
        b.digitalWrite(leds[i], state);
    }
}
{% endhighlight %}

这儿所使用的[BoneScript](https://github.com/jadonk/bonescript)是一个[Node.js](https://nodejs.org/)库，用于在BBB上操控硬件。[Cloud9](https://c9.io/)是一款在线集成开发环境，在BBB之外，可以直接在云端使用或者部署到私有环境。



# Blink External LED

* 真正的Hello World实验


## Wiring

* 实验器材
* 电阻计算
* 接线图

<figure>
  <a href="/images/photo/beaglebone/blink-led-fritzing.png">
    <img src="/images/photo/beaglebone/blink-led-fritzing.png" alt="Blah">
  </a>
</figure>

* 连接成功后的照片图


## Blink with Filesystem

* 插线图
* 命令行控制并解释
* Bash脚本闪亮

{% highlight bash %}
# Check PIN status
cat /sys/kernel/debug/gpio

ls /sys/class/gpio/
echo 30 > /sys/class/gpio/export

cat /sys/class/gpio/gpio30/direction
echo out > /sys/class/gpio/gpio30/direction

# Turn On -
echo 1 > /sys/class/gpio/gpio30/value

# Turn Off -
echo 0 > /sys/class/gpio/gpio30/value
{% endhighlight %}


{% highlight bash %}
#!/bin/bash

echo 30 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio30/direction

blink_on=0
while [ 1 ]; do
    if [ $blink_on = 0 ]; then
        blink_on=1
    else
        blink_on=0
    fi
    echo $blink_on > /sys/class/gpio/gpio30/value
    sleep 0.5
done
{% endhighlight %}


## Blink with Node.js

* 用BoneScript编写

{% highlight javascript %}
var b = require('bonescript');

var targetPin = "P9_11";
var state = b.LOW;

b.pinMode(targetPin, b.OUTPUT);
setInterval(toggle, 500);

function toggle() {
    if (state == b.LOW) { 
        state = b.HIGH; 
    }
    else { 
        state = b.LOW;
    }
    b.digitalWrite(targetPin, state);
}
{% endhighlight %}


## Blink with Python

* 用Python编写

{% highlight python %}
#!/usr/bin/env python

from Adafruit_BBIO import GPIO
from time import sleep

targetPin = "P9_11"
GPIO.setup(targetPin, GPIO.OUT)

status = GPIO.LOW

while True:
    if status == GPIO.LOW:
        status = GPIO.HIGH
    else:
        status = GPIO.LOW
    GPIO.output(targetPin, status)
    sleep(0.5)

GPIO.cleanup()
{% endhighlight %}





# Dim the LED

## Wiring



