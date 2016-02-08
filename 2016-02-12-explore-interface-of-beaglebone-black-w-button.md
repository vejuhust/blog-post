---
layout: post
title: Explore Interface of BeagleBone Black with Button
excerpt: "Read buttons: pull-up/down, polling, interrupt"
modified: 2016-02-12
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}






# Read the Button


## Pull-Up Resistor

* 搭建按钮
* 引入上拉概念

{% highlight bash %}
#!/bin/bash

GPIO_NUM=61 # has an internal pull-up resistor by default

echo "$GPIO_NUM" > /sys/class/gpio/export
printf "pin = gpio_%d\n" "$GPIO_NUM"

echo in > /sys/class/gpio/gpio"$GPIO_NUM"/direction
printf "direction = %s\n" $(cat /sys/class/gpio/gpio"$GPIO_NUM"/direction)

while [ 1 ]; do
    printf "[%s] input = %s\n" $(date '+%H:%M:%S.%N') $(cat /sys/class/gpio/gpio"$GPIO_NUM"/value)
    sleep 0.5
done
{% endhighlight %}


## Pull-Down Resistor

* 引入下拉概念

{% highlight bash %}
#!/bin/bash

GPIO_NUM=44 # has an internal pull-down resistor by default

echo "$GPIO_NUM" > /sys/class/gpio/export
printf "pin = gpio_%d\n" "$GPIO_NUM"

echo in > /sys/class/gpio/gpio"$GPIO_NUM"/direction
printf "direction = %s\n" $(cat /sys/class/gpio/gpio"$GPIO_NUM"/direction)

while [ 1 ]; do
    printf "[%s] input = %s\n" $(date '+%H:%M:%S.%N') $(cat /sys/class/gpio/gpio"$GPIO_NUM"/value)
    sleep 0.5
done
{% endhighlight %}


## Internal Pull-Up/Down Resistors

* 引入BBB内部上拉/下拉电路
* 考虑外部上拉/下拉电路与内部上拉/下拉相冲突的情况
* 外下拉内上拉，全部输出是1：下拉接9.11
* 外上拉内下拉，工作正常：上拉接9.23

{% highlight bash %}
#!/bin/bash

GPIO_NUM=30 # config-pin P9_11 in -> pin 28 (44e10870) 0000002f pinctrl-single 

echo "$GPIO_NUM" > /sys/class/gpio/export
printf "pin = gpio_%d\n" "$GPIO_NUM"

echo in > /sys/class/gpio/gpio"$GPIO_NUM"/direction
printf "direction = %s\n" $(cat /sys/class/gpio/gpio"$GPIO_NUM"/direction)

while [ 1 ]; do
    printf "[%s] input = %s\n" $(date '+%H:%M:%S.%N') $(cat /sys/class/gpio/gpio"$GPIO_NUM"/value)
    sleep 0.5
done
{% endhighlight %}


## Configure GPIO Pins

* pinmux设置法
* device tree概念
* 设置工具config-pin
* 下拉接9.11的修复：P9\_11 in 或 P9\_11 in-

{% highlight bash %}
echo cape-universaln > /sys/devices/bone_capemgr.*/slots
cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins | grep "pin 28 "
config-pin P9_11 in
cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins | grep "pin 28 "
{% endhighlight %}


{% highlight bash linenos %}
root@beaglebone:~# echo cape-universaln > /sys/devices/bone_capemgr.*/slots↩
root@beaglebone:~# cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins | grep "pin 28 "↩
pin 28 (44e10870) 00000037 pinctrl-single 
root@beaglebone:~# /var/lib/cloud9/press-button.sh↩
/var/lib/cloud9/press-button.sh: line 7: echo: write error: Device or resource busy
pin = gpio_30
direction = in
[18:22:21.852837181] input = 1
[18:22:22.408367889] input = 1
[18:22:22.963011931] input = 1
[18:22:23.518651889] input = 1
[18:22:24.074203556] input = 1
[18:22:24.629658348] input = 1
^C
root@beaglebone:~# config-pin P9_11 in↩
root@beaglebone:~# cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins | grep "pin 28 "↩
pin 28 (44e10870) 0000002f pinctrl-single 
root@beaglebone:~# /var/lib/cloud9/press-button.sh↩
/var/lib/cloud9/press-button.sh: line 7: echo: write error: Device or resource busy
pin = gpio_30
direction = in
[18:23:01.462888810] input = 0
[18:23:02.026213727] input = 0
[18:23:02.581415311] input = 1
[18:23:03.136452436] input = 0
[18:23:03.691387311] input = 0
[18:23:04.247814727] input = 1
[18:23:04.805659769] input = 0
^C
root@beaglebone:~# 
{% endhighlight %}






# Turn on the LED

## Push Switch by Polling

* 电路设计
* 通过轮询方式实现按压式开关

{% highlight bash %}
#!/bin/bash

SWITCH_GPIO=49
LIGHT_GPIO=7

echo "$SWITCH_GPIO" > /sys/class/gpio/export
printf "switch_pin = gpio_%d\n" "$SWITCH_GPIO"

echo in > /sys/class/gpio/gpio"$SWITCH_GPIO"/direction
printf "switch_direction = %s\n" $(cat /sys/class/gpio/gpio"$SWITCH_GPIO"/direction)

echo "$LIGHT_GPIO" > /sys/class/gpio/export
printf "light_pin = gpio_%d\n" "$LIGHT_GPIO"

echo out > /sys/class/gpio/gpio"$LIGHT_GPIO"/direction
printf "light_direction = %s\n" $(cat /sys/class/gpio/gpio"$LIGHT_GPIO"/direction)

while [ 1 ]; do
    light_status=$(cat /sys/class/gpio/gpio"$SWITCH_GPIO"/value)
    echo $light_status > /sys/class/gpio/gpio"$LIGHT_GPIO"/value
    printf "[%s] light = %s\n" $(date '+%H:%M:%S.%N') $light_status
    sleep 0.2
done
{% endhighlight %}


{% highlight python %}
#!/usr/bin/env python

from Adafruit_BBIO import GPIO
from time import sleep, time

switchPin = "P9_23"
lightPin = "P9_42"

GPIO.setup(switchPin, GPIO.IN)
GPIO.setup(lightPin, GPIO.OUT)

previouStatus = ""
timeStart = time()

def change_light_status(lightStatus):
    global previouStatus
    GPIO.output(lightPin, lightStatus)
    if lightStatus != previouStatus:
        print "[%9.5f] light = %s" % (time() - timeStart, lightStatus)
        previouStatus = lightStatus
    sleep(0.2)

while True:
    while GPIO.input(switchPin):
        change_light_status(GPIO.HIGH)
    change_light_status(GPIO.LOW)

GPIO.cleanup()
{% endhighlight %}


## Status Switch by Interrupt

底层的实现是基于Linux的[epoll_wait()](http://man7.org/linux/man-pages/man2/epoll_wait.2.html)系统调用。

{% highlight python %}
#!/usr/bin/env python

from Adafruit_BBIO import GPIO
from time import sleep, time

switchPin = "P9_23"
lightPin = "P9_42"

GPIO.setup(switchPin, GPIO.IN)
GPIO.setup(lightPin, GPIO.OUT)

previouStatus = ""
timeStart = time()

def change_light_status(lightStatus):
    global previouStatus
    GPIO.output(lightPin, lightStatus)
    if lightStatus != previouStatus:
        print "[%9.5f] light = %s" % (time() - timeStart, lightStatus)
        previouStatus = lightStatus
    sleep(0.2)

while True:
    GPIO.wait_for_edge(switchPin, GPIO.FALLING)
    change_light_status(GPIO.HIGH)
    GPIO.wait_for_edge(switchPin, GPIO.FALLING)
    change_light_status(GPIO.LOW)

GPIO.cleanup()
{% endhighlight %}

使用PyBBIO库，并加入最小间隔时间限制，避免闪烁灯泡。底层的实现是基于Linux的[epoll()](http://man7.org/linux/man-pages/man7/epoll.7.html)系统调用。

{% highlight python %}
#!/usr/bin/env python

from bbio import *
from time import time

switchPin = GPIO1_17
lightPin = GPIO0_7

def change_light_status():
    global timeStart
    global timePrevious
    timeCurrent = time()
    if timeCurrent - timePrevious > 0.1:
        toggle(lightPin)
        timePrevious = timeCurrent
        print "[%9.5f] light = %s" % (timeCurrent - timeStart, pinState(lightPin))

def setup():
    global timeStart
    global timePrevious
    pinMode(switchPin, INPUT, PULLDOWN)
    pinMode(lightPin, OUTPUT)
    attachInterrupt(switchPin, change_light_status, FALLING)
    timeStart = time()
    timePrevious = timeStart

def loop():
    delay(1000)

run(setup, loop)
{% endhighlight %}



