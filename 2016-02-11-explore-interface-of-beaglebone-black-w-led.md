---
layout: post
title: Explore Interface of BeagleBone Black with LED
excerpt: "It's all about LED: blink, turn on/off, dim"
modified: 2016-02-11
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}




# Blink the LED

* Hello World实验
* 实验器材
* 电阻计算

## Blink On-Board LED

* USR0 LED插图
* 用BoneScript编写

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


## Blink External LED with Bash

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


## Blink External LED with Node.js

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


## Blink External LED with Python

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



# Light the LED

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




# Dim the LED




