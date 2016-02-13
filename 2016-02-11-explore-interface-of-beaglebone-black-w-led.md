---
layout: post
title: Explore Interface of BeagleBone Black with LED
excerpt: "It's all about LED: blink, turn on/off, dim"
modified: 2016-02-11
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}




# Blink On-Board LED

## On-Board LED

* USR0 LED插图


## Environment

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



# Blink External LED

* Hello World实验


## Wiring

* 实验器材
* 电阻计算


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



