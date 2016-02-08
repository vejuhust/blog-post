---
layout: post
title: Explore Interface of BeagleBone Black with Display
excerpt: "Say it: 7-segment display or liquid-crystal display"
modified: 2016-02-15
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}





# Seven-Segment Display





# Liquid-Crystal Display

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


