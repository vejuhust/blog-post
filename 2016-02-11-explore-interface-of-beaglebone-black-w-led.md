---
layout: post
title: Explore Interface of BeagleBone Black with LED
excerpt: "It's all about LED: blink, turn on/off, dim"
modified: 2016-02-11
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}


Light-Emitting Diode (a.k.a. LED)中文叫作发光二极管，是一种能发光的半导体电子元件，在近些年被普遍用作照明用途之前，曾主要用作指示灯及显示板。这次我们使用LED目的就是来指示电路是否正确的接通，堪称“Hello World”实验。



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

* 用浏览器登录

<figure>
  <a href="/images/photo/beaglebone/webportal-screen.png">
    <img src="/images/photo/beaglebone/webportal-screen.png" alt="Blah">
  </a>
</figure>

* 按提示打开Cloud9
* 贴入脚本并运行，要有耐心

<figure>
  <a href="/images/photo/beaglebone/cloud9-screen.png">
    <img src="/images/photo/beaglebone/cloud9-screen.png" alt="Blah">
  </a>
</figure>

* BoneScript介绍
* 代码修改
* 验证环境没有问题

{% highlight javascript %}
var b = require('bonescript');

var state = b.LOW;

b.pinMode("USR0", b.OUTPUT);
b.pinMode("USR1", b.OUTPUT);
b.pinMode("USR2", b.OUTPUT);
b.pinMode("USR3", b.OUTPUT);
setInterval(toggle, 500);

function toggle() {
    if (state == b.LOW) { 
        state = b.HIGH; 
    }
    else { 
        state = b.LOW; 
    }
    b.digitalWrite("USR3", state);
}
{% endhighlight %}



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



