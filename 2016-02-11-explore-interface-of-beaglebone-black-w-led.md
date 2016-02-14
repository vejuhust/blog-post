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

如上图所示，BBB板载有5枚蓝色LED，以太网口左边的是Power LED，右边是User LED从3到0依次排开。他们默认的作用如下：

* PWR: BBB启动成功后会常亮；如果启动失败，BBB会在其闪烁后关机
* USR0: 展示来自Linux内核的心跳包，内核挂掉后将停止闪烁
* USR1: 亮起表示microSD卡正在被访问
* USR2: 亮起表示Linux内核没有处于idle状态
* USR3: 亮起表示板载eMMC正在被访问


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


## Trigger User LED

既然环境是正常的，我们不妨深入一点，进一步探究一下User LED。

通过SSH登录到BBB，或在BBB的Cloud9中打开Terminal窗口。进入`/sys/class/leds/`目录，我们可以看到这四枚“指蓝为绿”的LED：

{% highlight bash linenos %}
root@beaglebone:~# cd /sys/class/leds/↩
root@beaglebone:/sys/class/leds# ls -l↩
total 0
lrwxrwxrwx 1 root root 0 Jan  1  2000 beaglebone:green:usr0 -> ../../devices/ocp.3/gpio-leds.8/leds/beaglebone:green:usr0
lrwxrwxrwx 1 root root 0 Jan  1  2000 beaglebone:green:usr1 -> ../../devices/ocp.3/gpio-leds.8/leds/beaglebone:green:usr1
lrwxrwxrwx 1 root root 0 Jan  1  2000 beaglebone:green:usr2 -> ../../devices/ocp.3/gpio-leds.8/leds/beaglebone:green:usr2
lrwxrwxrwx 1 root root 0 Jan  1  2000 beaglebone:green:usr3 -> ../../devices/ocp.3/gpio-leds.8/leds/beaglebone:green:usr3
root@beaglebone:/sys/class/leds# 
{% endhighlight %}

因为当前在运行microSD中的固件，所以USR3很适合重新利用：

{% highlight bash linenos %}
root@beaglebone:~# cd /sys/class/leds/beaglebone\:green\:usr3/↩
root@beaglebone:/sys/class/leds/beaglebone:green:usr3# ls -l↩
total 0
-rw-r--r-- 1 root root 4096 Feb 14 17:16 brightness
lrwxrwxrwx 1 root root    0 Feb 14 18:11 device -> ../../../gpio-leds.8
-r--r--r-- 1 root root 4096 Feb 14 18:11 max_brightness
drwxr-xr-x 2 root root    0 Feb 14 18:11 power
lrwxrwxrwx 1 root root    0 Jan  1  2000 subsystem -> ../../../../../class/leds
-rw-r--r-- 1 root root 4096 Feb 14 17:16 trigger
-rw-r--r-- 1 root root 4096 Jan  1  2000 uevent
root@beaglebone:/sys/class/leds/beaglebone:green:usr3# cat trigger↩
none nand-disk mmc0 [mmc1] timer oneshot heartbeat backlight gpio cpu0 default-on transient 
root@beaglebone:/sys/class/leds/beaglebone:green:usr3# 
{% endhighlight %}

`/sys/class/leds/beaglebone:green:usr3`目录下的文件意义如下：

* brightness: 亮度，0表示关闭，大于等于1表示发光，读取获得数值，写入可以修改
* max_brightness: brightness的最大值，为255
* trigger: 触发方式，读取获得当前触发方式以所支持的触发方式，写入可以修改

我们可以看出USR3当前的trigger是mmc1(被用`[]`标注)，各个[trigger](https://www.kernel.org/doc/Documentation/leds/leds-class.txt)的作用如下：

* none: no-op，无trigger
* mmc0: 亮起表示microSD卡正在被访问，即USR1的默认trigger
* mmc1: 亮起表示板载eMMC正在被访问，即USR3的默认trigger
* timer: 计时器，亮起delay\_on毫秒后，熄灭delay\_off毫秒，周而复始
* [oneshot](https://www.kernel.org/doc/Documentation/leds/ledtrig-oneshot.txt): shot收到非空输入后，按delay\_on/delay\_off方式单次闪灯
* heartbeat: 展示来自Linux内核的心跳包，即USR0的默认trigger
* gpio: 亮起表示GPIO端口正在被访问
* cpu0: 亮起表示Linux内核没有处于idle状态，即USR2的默认trigger
* default-on: 常亮
* [transient](https://www.kernel.org/doc/Documentation/leds/ledtrig-transient.txt): activate收到非0输入后，亮点(state=1)/熄灭(state=0)灯duration毫秒


以timer为例，亮起200ms后，熄灭800ms，以1Hz的频率闪烁：

{% highlight bash %}
cd /sys/class/leds/beaglebone\:green\:usr3/
echo timer > trigger
echo 200 > delay_on
echo 800 > delay_off
{% endhighlight %}

或者以10Hz的频率闪烁：

{% highlight bash %}
echo timer > trigger
echo 50 > delay_on
echo 50 > delay_off
{% endhighlight %}

以oneshot为例，每次shot收到输入后亮起1s，熄灭100ms。但实际的频率闪烁为0.5Hz，因为shot在强制熄灭时收到的输入无作用。

{% highlight bash %}
echo oneshot > trigger
echo 1000 > delay_on
echo 100 > delay_off
while true; do echo 1 > shot; sleep 1; done
{% endhighlight %}

以transient为例，每次shot收到输入后亮起1s。因为没有强制熄灭的设置，此处会常亮。其间偶尔闪烁是由`echo`执行时间以及`sleep`精度问题造成的，改为`sleep 0.999`后闪烁间隔变大。

{% highlight bash %}
echo transient > trigger
echo 1000 > duration
echo 1 > state
while true; do echo 1 > activate; sleep 1; done
{% endhighlight %}

对了，这些是在RAM中的，重启之后一切恢复默认。



# Blink External LED

确认了开发环境正常后，我们来接上面包板，进行真正的“Hello World”实验啦。我们的目标是通过GPIO来控制LED的闪烁。


## Wiring

LED正向导通时的电阻并不大，直接接线会有短路烧毁BBB的风险。GPIO端口的电压为3.3V，最大输出电流为6mA，我们需要给LED串联一个330Ω的电阻。具体接线步骤如下：

1. 将BBB的P9_1端口与面包板的接地相连
2. 将LED的负极(即短引脚)与面包板的接地相连
3. 将LED的正极(即长引脚)与330Ω的电阻串联
4. 将330Ω的电阻的另一端与BBB的P9_15端口相连

<figure>
  <a href="/images/photo/beaglebone/blink-led-fritzing.png">
    <img src="/images/photo/beaglebone/blink-led-fritzing.png" alt="Breadboard Figure for 'Blink External LED'">
  </a>
</figure>

上图为接线示意图，下图为点亮LED后的实拍照片。

<figure>
  <a href="/images/photo/beaglebone/blink-led-photo.jpg">
    <img src="/images/photo/beaglebone/blink-led-photo.jpg" alt="Photo of Wiring for 'Blink External LED'">
  </a>
</figure>


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



