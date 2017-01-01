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

这是一个典型的[MVVM模式](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)，其中App Service(逻辑层)作为*Model*向作为*View*的View(视图层)发送数据用于展示，而后者又将被触发的事件发送给前者，这一切都是通过作为*View Model*的Native(系统层)传递的。

![Framework Inside App]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/overview-figure-2.png)


## View

小程序框架在[**View(视图层)**](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/)执行开发者编写的WXML和WXSS代码，并将其展示为WebView上的组件。

### Page Frame

小程序所使用的WebView被称为**Page Frame**，每个Page Frame占用1个线程，并独立初始化。每个小程序最多可以同时启动9个Page Frame线程，即最多5层的页面栈加上最多5个tabBar(第一个tabBar同属于页面栈最底层)。

打开页面时，Native(系统层)会额外预加载一个WebView，并用类似以下的HTML代码进行初始化。这个初始化过程花费大约100ms，加载在`<script/>`中的JavaScript代码包含小程序框架自身的代码以及被编译后的所有页面的WXML和WXSS代码。初始化完成后，这个WebView会被闲置，直到需要打开新的页面，Page Frame收到指令后会用大概100ms去执行`<script/>`中的代码将其指定的具体页面内容渲染在`<body/>`中，其过程无需请求额外资源。上述过程在后台执行完毕之后，才会将页面推到前台真正展现给用户。这一机制加速了小程序页面切换速度，保证其体验与传统Web App相比更加流畅。

{% highlight html %}
<html>
<head>
    <script>
        // initial scripts
    </script>
</head>
<body>
    <!-- content -->
</body>
</html>
{% endhighlight %}


### WXML

[WXML](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/)即**W**ei**X**in **M**arkup **L**anguage是微信基于XML和[{% raw %}{{ mustache }}{% endraw %}](https://mustache.github.io/)设计的用于描述小程序页面结构和绑定的标签语言。它的作用类似HTML，但具有算术运算、逻辑运算、模板和引用的能力。样例见下：

{% highlight xml %}
{% raw %}
<view wx:for="{{messages}}">{{item}}</view>
{% endraw %}
{% endhighlight %}

微信编译器会将开发者编写的WXML代码编译成JavaScript代码，为各个页面生成相对应的`generateFunc`，其输入是来自App Service的数据，输出则是对应的Virtual Tree，后者由经Virtual DOM算法将更新体现真实的DOM Tree上。


### WXSS

[WXSS](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxss.html)即**W**ei**X**in **S**tyle **S**heets是微信基于CSS设计的用于描述小程序页面样式的语言。它具有CSS大部分特性(不支持级联，以保护组件内部样式)，并在此基础上增加了自适应单位**RPX** (Responsive Pixel)以及`@import`语法导入外联样式表。样例见下：

{% highlight css %}
@import "common.wxss";
.foo__bar {
    padding: 20rpx;
}
{% endhighlight %}

微信编译器同样会将开发者编写的WXSS代码编译成JavaScript代码，微信客户端通过执行JavaScript代码来生成相应的CSS，会根据具体机型将rpx转化为相适应的px。


### Component



## App Service

### Route


## Life Cycle

### Inside App

### System-wide


## ProTip™
