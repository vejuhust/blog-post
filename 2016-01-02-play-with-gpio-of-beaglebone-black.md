---
layout: post
title: Explore Interface of BeagleBone Black by Example
excerpt: "My incomplete programming guide to play with hardware on BBB"
modified: 2016-01-02
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}


BeagleBone Black (a.k.a. BBB)


# Prepare the Board

在进一步实验之前，需要先准备好BBB的配件以及microSD卡。


## Accessories for BBB

如果选择以太网口连接BBB的话，网线之外，以下二者选其一用于供电：

* mini USB数据线
* 5V 1A直流电源适配器

在不差钱的情况下，还有其他配件可以考虑：

* microSD卡 - 至少4GB，以及读卡器，用于搭载或刷入操作系统
* USB Hub - 以扩展BBB上唯一的USB接口
* USB转TTL-232数据线 - 在无网络的情况，通过串口连接
* 保护壳 - 能一定程度上防摔和防静电
* Micro HDMI视频线 - 用于连接支持HDMI的显示器
* 键鼠套装 - 如果需要直接作为桌面机器使用


## Install Debian on microSD Card

BBB自带的eMMC中载有Ångström或Debian系统，如果不满足于eMMC的容量或者想使用其他操作系统，我们需要将其写入microSD卡。

先去BeagleBone官方网站<http://beagleboard.org/latest-images>寻找合适的Debian镜像，这次我选择的是2015年11月12日发布的Debian 7.9版本。用任意工具直接下载即可，考虑到国内下载比较慢，不妨用海外的机器下载再复制回来：

{% highlight bash %}
# On Linux
wget http://builds.beagleboard.org/images/master/08132bf0d0cb284d1148c5d329fe3c8e1aaee44d/bone-debian-7.9-lxde-4gb-armhf-2015-11-12-4gb.img.xz

# On Mac OS X
curl -O http://builds.beagleboard.org/images/master/08132bf0d0cb284d1148c5d329fe3c8e1aaee44d/bone-debian-7.9-lxde-4gb-armhf-2015-11-12-4gb.img.xz
{% endhighlight %}

为了避免后续步骤可能的失败，拿到文件后先校验一下：

{% highlight bash %}
# On Linux
echo "f6e67ba01ff69d20f2c655f5e429c3e6c2398123bcd3d8d548460c597275d277  bone-debian-7.9-lxde-4gb-armhf-2015-11-12-4gb.img.xz" | sha256sum -c

# On Mac OS X
echo "f6e67ba01ff69d20f2c655f5e429c3e6c2398123bcd3d8d548460c597275d277  bone-debian-7.9-lxde-4gb-armhf-2015-11-12-4gb.img.xz" | shasum -a 256 -c
{% endhighlight %}

确认文件正确后，使用`unxz`命令对其解压缩。Ubuntu已包含此工具，Mac OS X需要从[Mac OS X Packages](http://macpkg.sourceforge.net)开源项目中获取。

{% highlight bash %}
unxz bone-debian-7.9-lxde-4gb-armhf-2015-11-12-4gb.img.xz 
{% endhighlight %}

准备好一张至少4GB容量的microSD卡，在插入机器前后各运行一次`df -h`命令，以通过对比结果找出microSD卡的设备名，这次是/dev/disk9s1。然后将其卸载：

{% highlight bash %}
sudo diskutil unmount /dev/disk9s1
{% endhighlight %}

使用`dd`命令将镜像写入microSD卡中，注意这儿应使用原始设备名，即/dev/rdisk9而不是/dev/disk9s1：

{% highlight bash %}
sudo dd bs=2m if=bone-debian-7.9-lxde-4gb-armhf-2015-11-12-4gb.img of=/dev/rdisk9
{% endhighlight %}

写入完成后将其弹出：

{% highlight bash %}
sudo diskutil eject /dev/rdisk9
{% endhighlight %}

取出microSD卡并插入关机状态的BBB中，按住Boot Switch按钮并通电开机，当BBB上的两个蓝色LED点亮时松开即可。/* 在我的BBB上，不按按钮也能正确从microSD卡加载系统。 */查询到BBB的IP地址，用SSH直接登陆，默认的用户名是`debian`，密码是`temppwd`。登陆后记得使用`passwd`命令修改。

## Expand File System Partition

如果使用的microSD卡不止4GB，那么我们需要在BBB上调整它的分区表来充分利用剩余的空间。用`fdisk`的操作记录如下，输入部分均用`↩︎`标出：

{% highlight bash %}
root@beaglebone:~# fdisk /dev/mmcblk0↩

Command (m for help): p↩

Disk /dev/mmcblk0: 15.9 GB, 15931539456 bytes
4 heads, 16 sectors/track, 486192 cylinders, total 31116288 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

        Device Boot      Start         End      Blocks   Id  System
/dev/mmcblk0p1   *        2048      198655       98304    e  W95 FAT16 (LBA)
/dev/mmcblk0p2          198656     6963199     3382272   83  Linux

Command (m for help): d↩
Partition number (1-4): 2↩

Command (m for help): n↩
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p↩
Partition number (1-4, default 2): 2↩
First sector (198656-31116287, default 198656): ↩
Using default value 198656
Last sector, +sectors or +size{K,M,G} (198656-31116287, default 31116287): ↩
Using default value 31116287

Command (m for help): p↩

Disk /dev/mmcblk0: 15.9 GB, 15931539456 bytes
4 heads, 16 sectors/track, 486192 cylinders, total 31116288 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

        Device Boot      Start         End      Blocks   Id  System
/dev/mmcblk0p1   *        2048      198655       98304    e  W95 FAT16 (LBA)
/dev/mmcblk0p2          198656    31116287    15458816   83  Linux

Command (m for help): w↩
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
root@beaglebone:~# 
{% endhighlight %}

重新启动后再次SSH登陆，调整文件系统以使用更大的分区：

{% highlight bash %}
sudo resize2fs /dev/mmcblk0p2
{% endhighlight %}

至此，文件系统已经调整完毕，可以用`df -h`命令查看结果：

{% highlight bash %}
root@beaglebone:~# df -h↩
Filesystem      Size  Used Avail Use% Mounted on
rootfs           15G  1.9G   12G  14% /
udev             10M     0   10M   0% /dev
tmpfs           100M  552K   99M   1% /run
/dev/mmcblk0p2   15G  1.9G   12G  14% /
tmpfs           249M     0  249M   0% /dev/shm
tmpfs           249M     0  249M   0% /sys/fs/cgroup
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           100M     0  100M   0% /run/user
{% endhighlight %}



# GPIO Pins

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


{% highlight bash %}
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

{% highlight bash %}
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


# PM2.5 Sensor

## UART: E-E-c-c-h-h-o-o

{% highlight bash %}
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

