---
layout: post
title: Explore Interface of BeagleBone Black by Example
excerpt: "My incomplete programming guide to play with hardware on BBB"
modified: 2016-01-07
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}


与[Raspberry Pi](https://www.raspberrypi.org/)和[Arduino](https://www.arduino.cc/)相比，[BeagleBone Black](http://beagleboard.org/black) (a.k.a. BBB)的一大优势就是扩展接口相对较多。这些PC或Mac所不具备的扩展接口可以让你与各类嵌入式设备交互，更加接近物理世界，获得与软件编程所不同的体验和乐趣。而这篇笔记就是以多个小实验来探讨如何通过BBB来操作各类设备的。

如果说这些在硬件上的摸索能给一位软件工程师带来什么经验的话，或许就是以下几条了：

:thought_balloon:  **Tips**     
1. 小心静电。     
2. 通电之前检查连线，检查，检查，再检查！     
3. 仔细阅读正确的datasheet。     
4. [StackOverflow](http://electronics.stackexchange.com/)并不是万能的。     
5. 与追着log去debug软件相比，硬件是麻烦而痛苦。     
{: .notice}


# Headers

## 总体概况

* 模式介绍
* pdf文件

## GPIO
* GPIO的插图
* 几种GPIO名称的读法



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




# 电位器 Potentiometer




# 发声 Buzzer




# 收声 Microphone




# 活人 Human Proximity Sensor




# 光强 Digital Light Sensor 




# 继电器 Relay




# Humidity Sensor

## Hardware

* 传感器
* 连线图

## Weather Station

* 使用Adafruit_DHT的库
* 读取数据后通过HTTP HEAD发送到服务器

{% highlight python %}
#!/usr/bin/env python

import Adafruit_DHT
from time import time
from datetime import datetime
from httplib import HTTPConnection
from threading import Timer

def generate_report_string(date, temperature, humidity, time_lapsed):
    report_string = "{0:s};{1:0.2f};{2:0.2f};{3:0.3f};".format(date.isoformat(), temperature, humidity, time_lapsed)
    return report_string

def read_weather_status():
    date = datetime.now()
    time_start = time()
    humidity, temperature = Adafruit_DHT.read_retry(Adafruit_DHT.DHT11, "P9_12")
    return date, temperature, humidity, time() - time_start

def submit_weather_status():
    status_measure = False
    date, temperature, humidity, time_lapsed = read_weather_status()
    if humidity is not None and temperature is not None:
        headers = { "User-Agent": generate_report_string(date, temperature, humidity, time_lapsed)}
        connection = HTTPConnection("service.yewei.me")
        connection.request("HEAD", "/weather", headers = headers)
        status_measure = True
    return status_measure

def loop():
    Timer(30, loop).start()
    submit_weather_status()

if __name__ == '__main__':
    loop()
{% endhighlight %}


## Weather Channel

* 服务器清洗数据
* 服务器展示结果

{% highlight bash linenos %}
root@yewei-prod-hk:~# tail -f /var/log/nginx/access.log | grep -i "/weather"↩
123.120.42.45 - - [11/Jan/2016:04:11:15 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:09:44.398996;21.00;20.00;3.055;"
123.120.42.45 - - [11/Jan/2016:04:11:43 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:10:14.928819;21.00;20.00;0.526;"
123.120.42.45 - - [11/Jan/2016:04:12:19 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:10:45.458968;21.00;20.00;5.581;"
123.120.42.45 - - [11/Jan/2016:04:12:45 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:11:15.988912;21.00;20.00;0.526;"
123.120.42.45 - - [11/Jan/2016:04:13:15 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:11:46.521053;21.00;20.00;0.526;"
123.120.42.45 - - [11/Jan/2016:04:13:46 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:12:17.054274;21.00;20.00;0.526;"
123.120.42.45 - - [11/Jan/2016:04:14:16 +0000] "HEAD /weather HTTP/1.1" 404 0 "-" "2016-01-11T04:12:47.584879;21.00;20.00;0.527;"
{% endhighlight %}

{% highlight bash %}
cat /var/log/nginx/access.log | grep -i "/weather" | cut -d'"' -f6 | cut -d';' -f1-3 --output-delimiter=',' | sort > temp_humid.csv
{% endhighlight %}

{% highlight text %}
2016-01-12T04:56:07.936775,19.00,21.00
2016-01-12T04:56:38.468112,19.00,21.00
2016-01-12T04:57:09.001872,19.00,21.00
2016-01-12T04:57:39.532041,18.00,22.00
2016-01-12T04:58:10.062362,18.00,22.00
2016-01-12T04:58:40.595329,18.00,22.00
{% endhighlight %}

更多在线数据存储、处理、展示的选项：

* Carriots <https://www.carriots.com/>
* ioBridge <https://www.iobridge.com/>
* ThingSpeak <https://thingspeak.com/>
* XOBXOB <http://www.xobxob.com/>
* SensorCloud <http://www.sensorcloud.com/>


## 升级硬件



# PM2.5 Sensor

## UART: E-E-c-c-h-h-o-o

{% highlight bash linenos %}
root@beaglebone:~# ls /dev/ttyO*↩
/dev/ttyO0
root@beaglebone:~# ls /lib/firmware/*UART*↩
/lib/firmware/ADAFRUIT-UART1-00A0.dtbo	/lib/firmware/ADAFRUIT-UART2-00A0.dtbo	/lib/firmware/ADAFRUIT-UART4-00A0.dtbo	/lib/firmware/ADAFRUIT-UART5-00A0.dtbo
root@beaglebone:~# echo ADAFRUIT-UART4 > /sys/devices/bone_capemgr.9/slots↩
root@beaglebone:~# cat /sys/devices/bone_capemgr.9/slots↩
 0: 54:PF--- 
 1: 55:PF--- 
 2: 56:PF--- 
 3: 57:PF--- 
 4: ff:P-O-L Bone-LT-eMMC-2G,00A0,Texas Instrument,BB-BONE-EMMC-2G
 5: ff:P-O-L Bone-Black-HDMI,00A0,Texas Instrument,BB-BONELT-HDMI
 7: ff:P-O-L Override Board Name,00A0,Override Manuf,ADAFRUIT-UART4
root@beaglebone:~# ls /dev/ttyO*↩
/dev/ttyO0  /dev/ttyO4
root@beaglebone:~# 
{% endhighlight %}

{% highlight bash %}
apt-get update
apt-get install -y minicom
minicom -b 9200 -o -D /dev/ttyO4
{% endhighlight %}

{% highlight text %}
Welcome to minicom 2.6.1                                                                                           
                                                                                                                   
OPTIONS: I18n                                                                                                      
Compiled on Feb 11 2012, 18:45:56.                                                                                 
Port /dev/ttyO4                                                                                                    
                                                                                                                   
Press CTRL-A Z for help on special keys                                                                            
                                                                                                                   
hheelllloo,,  wwoorrlldd!!                                                                                         
                                                                                                                   
{% endhighlight %}


## Sensor Specification

* 说明传感器插线方法
* 传感器接线
* 传感器数据说明


## First Attempt

{% highlight python %}
#!/usr/bin/env python

from serial import Serial
from os import system

system("echo BB-UART4 > /sys/devices/bone_capemgr.9/slots")
system("ls /dev/ttyO*")

port = Serial('/dev/ttyO4', baudrate=9600, timeout=2.0)

def read_message():
    raw_bytes = port.read(32)
    raw_values = [ ord(raw_char) for raw_char in raw_bytes ]
    for index, raw_value in enumerate(raw_values):
        if index % 2 == 1:
            real_value = raw_value + (previous_value << 8)
            suffix_string = "{0:6d}".format(real_value)
        else:
            previous_value = raw_value
            suffix_string = ""
        print "{0:02d} : {1:02x} | {1:3d} | {2:s}".format(index + 1, raw_value, suffix_string)

read_message()
{% endhighlight %}

{% highlight text %}
/dev/ttyO0  /dev/ttyO4
01 : 42 |  66 | 
02 : 4d |  77 |  16973
03 : 00 |   0 | 
04 : 1c |  28 |     28
05 : 00 |   0 | 
06 : 19 |  25 |     25
07 : 00 |   0 | 
08 : 20 |  32 |     32
09 : 00 |   0 | 
10 : 24 |  36 |     36
11 : 00 |   0 | 
12 : 17 |  23 |     23
13 : 00 |   0 | 
14 : 1f |  31 |     31
15 : 00 |   0 | 
16 : 24 |  36 |     36
17 : 0f |  15 | 
18 : 93 | 147 |   3987
19 : 04 |   4 | 
20 : 5b |  91 |   1115
21 : 00 |   0 | 
22 : a9 | 169 |    169
23 : 00 |   0 | 
24 : 0c |  12 |     12
25 : 00 |   0 | 
26 : 00 |   0 |      0
27 : 00 |   0 | 
28 : 00 |   0 |      0
29 : 71 | 113 | 
30 : 00 |   0 |  28928
31 : 03 |   3 | 
32 : 89 | 137 |    905
{% endhighlight %}


## Serious Implementation

* UART初始化
* struct的unpack来读取数据

{% highlight python %}
#!/usr/bin/env python

from Adafruit_BBIO import UART
from serial import Serial
from time import sleep
from struct import unpack
from sys import exit
from os import _exit

uart_number = 4
message_length = 32
baud_rate = 9600
read_timeout_sec = 3
read_interval_sec = 0.5

def decode_message(raw_bytes):
    # Unpack data from message
    data_format = ">" + "H" * (message_length >> 1)
    data_list = unpack(data_format, raw_bytes)
    [ header, _, 
      tsi_pm10_ugm, tsi_pm25_ugm, tsi_pm100_ugm, 
      std_pm10_ugm, std_pm25_ugm, std_pm100_ugm, 
      cnt_pd_3um, cnt_pd_5um, cnt_pd_10um, cnt_pd_25um, cnt_pd_50um, cnt_pd_100um,
      _, checksum ] = data_list
    # Compute checksum
    check_format = ">" + "B" * (message_length - 2) + "xx"
    check_list = unpack(check_format, raw_bytes)
    # Publish
    if header == 0x424d and checksum == sum(value for value in check_list):
        print data_list
    else:
        print "failed to validate"

def read_device(serial_device):
    # Check if device is open for read
    while not serial_device.is_open:
        serial_device.open()
    print "device %s is open" % serial_device.name
    # Read and decode messages from device
    while True:
        bytes_count = serial_device.in_waiting
        while bytes_count >= message_length:
            message_bytes = serial_device.read(message_length)
            decode_message(message_bytes)
            bytes_count -= message_length
        sleep(read_interval_sec)

if __name__ == '__main__':
    UART.setup("UART%d" % uart_number)
    serial_device = Serial("/dev/ttyO%d" % uart_number, baudrate=baud_rate, timeout=read_timeout_sec)    
    try:
        read_device(serial_device)
    except KeyboardInterrupt:
        serial_device.close()
        print 'goodbye!'
        try:
            exit(0)
        except SystemExit:
            _exit(0)
{% endhighlight %}

{% highlight text %}
device /dev/ttyO4 is open
(16973, 28, 45, 60, 73, 32, 46, 61, 7047, 2000, 320, 36, 12, 8, 28928, 1099)
(16973, 28, 45, 60, 72, 32, 46, 61, 7092, 2004, 318, 32, 10, 6, 28928, 1137)
(16973, 28, 46, 61, 73, 33, 46, 61, 7197, 2026, 303, 34, 10, 6, 28928, 1000)
(16973, 28, 45, 60, 72, 32, 46, 61, 7080, 1985, 310, 34, 10, 6, 28928, 1100)
(16973, 28, 44, 59, 71, 32, 45, 60, 7008, 1962, 304, 36, 10, 6, 28928, 996)
(16973, 28, 43, 59, 69, 31, 45, 59, 6861, 1911, 310, 36, 6, 4, 28928, 1048)
^Cgoodbye!
{% endhighlight %}




# 七段数码管 Seven-Segment Display





# LCD

## 芯片及接口

## GPIO插线方法

* 调节屏幕对比度的电位器不可以省

{% highlight python %}
#!/usr/bin/python
# Example using a character LCD connected to a Raspberry Pi or BeagleBone Black.

from time import sleep
from Adafruit_CharLCD import Adafruit_CharLCD

# BeagleBone Black configuration:
lcd_rs        = 'P8_8'
lcd_en        = 'P8_10'
lcd_d4        = 'P8_18'
lcd_d5        = 'P8_16'
lcd_d6        = 'P8_14'
lcd_d7        = 'P8_12'
lcd_backlight = None

# Define LCD column and row size for 16x2 LCD.
lcd_columns = 16
lcd_rows    = 2

# Initialize the LCD using the pins above.
lcd = Adafruit_CharLCD(lcd_rs, lcd_en, lcd_d4, lcd_d5, lcd_d6, lcd_d7, lcd_columns, lcd_rows, lcd_backlight)

lcd.create_char(0, [0x0, 0x0, 0xa, 0x1f, 0x1f, 0xe, 0x4, 0x0])
lcd.create_char(1, [0x0, 0x0, 0xa, 0x15, 0x11, 0xa, 0x4, 0x0])

lcd.clear()
lcd.message('\x00 I Love You! \x00\n \x01\x00\x01 \xB1\xB2\xBC\xC3\xD9 \x7E\x5F\x7F')

sleep(1)
lcd.move_right()
{% endhighlight %}

## I2C转换方法

## USB转换方法




# 实时时钟 Real Time Clock


