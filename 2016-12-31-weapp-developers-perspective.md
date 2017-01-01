---
layout: single
title: "WeApp in WeChat: A Developer's Perspective"
excerpt: "Introduction to framework of WeApp in WeChat, tips & tricks for developers."
modified: 2016-12-31
tags: [wechat, develop, framework]
comments: true
---

为了更有效地进行微信小程序的开发，我参加了微信商学院11月23日在北京和12月20日在上海举办的两场开发者培训班，也与同事合作进行了两场公司内部的分享。这篇文章记录了对微信小程序框架的解析(基于微信客户端[iOS 6.5.2](https://itunes.apple.com/us/app/wechat/id414478124?mt=8)和[Android 6.3.32](https://play.google.com/store/apps/details?id=com.tencent.mm)版本)，以及在开发实战中总结的技巧。部分内容根据微信工程师胡浩的讲课整理而成。


## Overview

作为基于且依赖微信的[小程序框架](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/MINA.html)，其目标是通过类Web技术为开发者在微信中开发出媲美原生App的应用提供了一整套解决方案。

小程序框架在微信客户端上由三部分组成：**View(视图层)**、**App Service(逻辑层)**和**Native(系统层)**。View和App Service构成了一个单向的数据绑定系统，在客户端内部通过Data和Event交互。App Service通过JSBridge与Native通讯调用微信客户端的原生能力。小程序与外部服务的通讯全部由Native承担——当用户第一次打开小程序或更新的时候，Native会从腾讯的CDN下载小程序完整的package；小程序运行过程中与开发者服务器的通讯也同样被Native代理。示意图见下：

![Framework Overview]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/overview-figure-1.png)

参照下图，具体观其内部：

* **View(视图层)**起到了浏览器的作用，来展示各个页面。
* **App Service(逻辑层)**则承担了服务器的部分职责，提供本地存储，并支持离线功能：
  - Manager负责管理数据、页面的生命周期、事件的分发、路由跳转。
  - API基于JSSDK演化而来，通过JSBridge调用系统层的原生能力。
* **Native(系统层)**主要起到了桥梁的作用，视图层和逻辑层的交互以及对微信客户端原生能力的调用都是通过系统层进行连接。

![Framework Inside App]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/overview-figure-2.png)


## View

### Page Frame

### WXML

### WXSS

### Component



## App Service

### Route


## Life Cycle

### Inside App

### System-wide


## ProTip™
