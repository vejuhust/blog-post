---
layout: post
title: Explore Interface of BeagleBone Black with Sensor
excerpt: "Sense the world: people, light, humidity, heat, air"
modified: 2016-02-14
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}




# 活人 Human Proximity Sensor




# 光强 Digital Light Sensor 





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




