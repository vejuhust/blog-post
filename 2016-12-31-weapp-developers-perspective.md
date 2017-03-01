---
layout: single
title: "WeApp in WeChat: A Developer's Perspective"
excerpt: "Introduction to framework of WeApp in WeChat, tips & tricks for developers."
modified: 2017-01-02
tags: [wechat, develop, framework]
comments: true
---


微信小程序可以看做是微信对其自身平台上Web App(俗称H5应用)进行规范化的措施，它通过流量、性能和效率吸引开发者，同时满足了微信对内容审核和生态建立的需求。

2016年末，我参加了微信商学院11月23日在北京和12月20日在上海举办的两场开发者培训班，也与同事合作进行了两场公司内部的分享。这篇文章记录了对微信小程序框架的解析(基于微信客户端[iOS 6.5.2](https://itunes.apple.com/us/app/wechat/id414478124?mt=8)和[Android 6.3.32](https://play.google.com/store/apps/details?id=com.tencent.mm)版本)，以及在开发实战中的经验总结。部分内容根据微信工程师胡浩的讲课整理而成。


## 概况 Overview

作为基于且依赖微信的[小程序框架](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/MINA.html)，其目标是通过类Web技术为开发者在微信中开发出媲美原生App的应用提供了一整套解决方案。

小程序框架在微信客户端上由三部分组成：**View(视图层)**、**App Service(逻辑层)**和**Native(系统层)**。View和App Service构成了一个单向的数据绑定系统，在微信客户端内部通过Data和Event交互。App Service通过WeixinJSBridge与Native通讯以调用微信客户端的原生能力。小程序与外部服务的通讯全部由Native承担——当用户第一次打开小程序或更新的时候，Native会从腾讯的CDN下载小程序完整的package；小程序运行过程中与开发者服务器的进行通讯也同样由Native转发。示意图见下：

![Framework Overview]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/overview-1.jpg)

参照下图，具体观其内部：

* **View(视图层)**起到了浏览器的作用，来展示小程序的各个页面。
* **App Service(逻辑层)**承担了服务器的部分职责，提供本地存储，支持离线，内部由两块构成：
  - Manager负责管理数据、页面的生命周期、事件的分发、路由跳转。
  - [API](https://mp.weixin.qq.com/debug/wxadoc/dev/api/)基于[微信JS-SDK](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html)演化而来，通过WeixinJSBridge与系统层通讯，以调用微信客户端的原生能力。
* **Native(系统层)**主要起到了桥梁的作用，视图层和逻辑层的交互以及对微信客户端原生能力的调用都是通过系统层进行连接。

这是一个典型的[MVVM模式](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)，其中App Service(逻辑层)作为*Model*向作为*View*的View(视图层)发送数据用于展示，而后者又将被触发的事件发送给前者，这一切都是通过作为*View Model*的Native(系统层)传递的。

![Framework Inside App]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/overview-2.jpg)


## View

小程序框架在[**View(视图层)**](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/)执行开发者编写的WXML和WXSS代码，并将其展示为WebView上的组件。

### Page Frame

小程序所使用的WebView被称为**Page Frame**，每个Page Frame占用1个线程，并独立初始化。每个小程序最多可以同时启动9个Page Frame线程，即最多5层的页面栈加上最多5个tabBar(第一个tabBar同属于页面栈最底层)。

打开页面时，Native(系统层)会额外预加载一个WebView，并用类似以下的HTML代码进行初始化。这个初始化过程花费大约100ms，加载在`<script/>`中的JavaScript代码包含小程序框架自身的代码以及被编译后的所有页面的WXML和WXSS代码。初始化完成后，这个WebView会被闲置，直到需要打开新的页面，Page Frame收到指令后会去执行`<script/>`中的代码并结合App Service中定义的初始数据将其指定的具体页面内容渲染在`<body/>`中，其过程无需请求额外资源，这一步骤同样需要100ms时间。上述过程在后台执行完毕之后，才会将页面推到前台真正展现给用户。Page Frame机制提高了小程序页面的切换速度，保证其体验与传统Web App相比更加流畅。

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

微信参考[Polymer](https://www.polymer-project.org/1.0/)设计了[WX Component](https://mp.weixin.qq.com/debug/wxadoc/dev/component/)组件系统为开发者提供了基础组件，以便通过组合的方式进行快速开发。这种设计隔离了开发者的代码和底层具体的渲染方式，方便未来移植到新的平台，并支持自定义事件。

大部分组件是基于HTML在WebView内直接实现的，小部分组件的实现是在WebView层上覆盖了Native层(目前有下列四个组件是这样实现的)。因而微信小程序并不再是纯粹的Web App，而是一种Web+Native的Hybrid应用。

* `<canvas/>` - [画布组件](https://mp.weixin.qq.com/debug/wxadoc/dev/component/canvas.html)
* `<map/>` - [地图组件](https://mp.weixin.qq.com/debug/wxadoc/dev/component/map.html)
* `<video/>` - [视频组件](https://mp.weixin.qq.com/debug/wxadoc/dev/component/video.html)
* `<textarea/>` - [多行输入框](https://mp.weixin.qq.com/debug/wxadoc/dev/component/textarea.html)


## App Service

[**App Service(逻辑层)**](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/)是小程序框架的基础，它执行开发者用JavaScript编写的业务逻辑代码，并帮助View(视图层)与Native(系统层)交互。它还承担了数据绑定、事件分发、路由管理和生命周期管理的职责。开发者需要在App Service中编写[`App()`](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/app.html)和[`Page()`](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html)方法来完成对小程序自身和每个页面的注册。

### Binding

[事件绑定](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/event.html)的一个简单实例——用户点击按钮导致其文字、样式和图标展示发生变化。页面部分的WXML代码如下：

{% highlight xml %}
{% raw %}
<view class="button-wrapper">
    <button type="{{buttonType}}" size="mini" loading="{{loading}}" bindtap="setLoading">
        {{buttonText}}
    </button>
</view>
{% endraw %}
{% endhighlight %}

相应Page定义中声明的事件处理函数如下：

{% highlight javascript %}
Page({
    data: {
        buttonType: 'primary',
        buttonText: 'Load',
        loading: false,
    },
    setLoading: function(event) {
        var isLoading = this.data.loading;
        var newData = {
            loading: !isLoading,
        };
        if (isLoading) {
            newData["buttonText"] = "Load";
            newData["buttonType"] = "primary";
        } else {
            newData["buttonText"] = "Cancel";
            newData["buttonType"] = "warn";
        }
        console.log(isLoading, event);
        this.setData(newData);
    },
});
{% endhighlight %}


### Route

App Service负责对小程序中所有页面路由的管理，路由的触发方式以及页面生命周期函数参考[文档](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html#%E9%A1%B5%E9%9D%A2%E7%9A%84%E8%B7%AF%E7%94%B1)。有两点需要注意：

1. 小程序页面栈最高限制为五层，即不能同时打开超过五层的页面，要求开发者不能在小程序中设计过多层级。
2. App Service总共只有有一个线程，它会按顺序执行执行生命周期函数。耗时较长的生命周期函数并不会堵塞View渲染页面，但会造成App Service中其他事件的处理被延后。


## Life Cycle

从两个方面阐述小程序的生命周期方面，一是小程序在微信客户端内运行的生命周期，二是小程序从开发者提交到用户加载的全过程。

### Inside App

![App Life Cycle (Animation)]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/app-life-cycle.gif)

以上动图展示了一个小程序在微信客户端内从开启到关闭的全过程：

1. Native线程是微信客户端本身，它通过Launch事件启动App Service和View两个线程：
    * App Service线程调用注册在`App()`函数中的`onLaunch()`函数
    * View线程开始初始化过程，即进行Page Frame的公共库和所有页面的WXML和WXSS代码的注入
2. 用户打开小程序后，Native会发送Route事件以打开首页：
    * 在App Service中将Page进行实例化：
        * 将页面的初始化数据发送给View
        * 依次执行注册在`Page()`函数中的`onLoad()`和`onShow()`函数
    * View进行第二次初始化过程，即通知Page Frame渲染具体的页面：
        * 初始化完成后，开始渲染来自Page的初始化数据
        * 渲染完成后，会将完整的首页展示给用户
        * 并通知Page渲染已完成，让其调用`onReady()`函数
3. 当用户产生点击一类的输入，操作会以Event的形式发送给App Service，调用开发者定义的事件处理函数，执行后可能会将部分数据发回给View进行再次渲染
4. 当页面发生跳转时，Native再次发送一个Route事件给App Service和View：
    * App Service中当前的Page会进入隐藏的状态，调用`onHide()`函数，同时会加载另一个Page
    * Page Frame会加载另一个已经初始化的View线程
5. 如果Route事件导致返回到第一个页面，第一个页面将再次展现并调用App Service中对应Page中的`onShow()`函数
6. 页面被关闭时，App Service中对应的Page会调用`onUnload()`函数
7. 当用户按Home键或左上角的退出，小程序并没有被真正的关闭，而是会进入后台并调用App Service自身的`onHide()`函数；当用户再此进入小程序时，会调用App Service自身的`onShow()`函数。当小程序进入后台一定时间之后，或占用系统资源过高时，该小程序的View和App Service线程才会被真正销毁。

完整的生命周期示意图见下：

![App Life Cycle]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/app-life-cycle.png)


### Distribution Process

开发者通过[微信web开发者工具(DevTools)](https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/devtools.html)提交所编写的小程序代码，开发者工具会将工程目录下的全部文件打包上传至微信后台服务器。微信后台服务器会将wxml、wxss和js文件都编译并打包为JavaScript，其中wxml和wxss文件打包后会合并，json配置文件仅做打包。用户打开小程序时，微信客户端会从腾讯的CDN下载并加载编译后的小程序，全部wxml和wxss文件的内容最终进入View，所有js文件的内容进入App Server，json配置被Native所用。过程图解见下：

![Distribution Process]({{ site.url }}{{ site.baseurl }}/images/photo/weapp-develop/distribute-process.jpg)


## ProTip™

总结一下这几周以来发现的技巧和趟过的坑。

### Tips

* 微信小程序的JavaScript运行环境，根据平台分别是：
  - iOS: [JavaScriptCore](https://developer.apple.com/reference/javascriptcore)
  - Android: [X5 JavaScript解析器](http://x5.tencent.com/)
  - DevTools: [NW.js](https://nwjs.io/)
* Page Frame所涉及的[Virtual DOM](https://medium.com/cardlife-app/what-is-virtual-dom-c0ec6d6a925c)算法来自[React](https://facebook.github.io/react/)，有相当多的[第三方实现](http://vdom-benchmark.github.io/vdom-benchmark/)。
* 小程序支持ECMAScript 6的语法，但不支持它的函数。
* 在微信客户端上运行的小程序必须使用HTTPS与开发者服务器通讯，[TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)版本要求为1.2。


### Tricks

* DevTools中包含了命令行版本的小程序WXML和WXSS编译器(即`wcc`和`wcsc`)，可以在工具的Console中执行`openVendor()`命令打开该目录，或者直接访问：
  - Windows: `{% raw %}%PROGRAMFILES(X86)%\Tencent\微信web开发者工具\package.nw\app\dist\weapp\onlinevendor\{% endraw %}`
  - macOS: `{% raw %}/Applications/wechatwebdevtools.app/Contents/Resources/app.nw/app/dist/weapp/onlinevendor/{% endraw %}`
* DevTools使用[NW.js](https://nwjs.io/)编写而成，可以通过修改相关.js文件直接对其hack(需要重启DevTools)，以0.11.122100版本为例，程序目录在：
  - Windows: `{% raw %}%PROGRAMFILES(X86)%\Tencent\微信web开发者工具\package.nw\app\{% endraw %}`
  - macOS: `{% raw %}/Applications/wechatwebdevtools.app/Contents/Resources/app.nw/app/{% endraw %}`


### Traps

* 使用Native实现的混合组件(例如，`<canvas/>`、`<map/>`等)展现时是凌驾于WebView层之上的，因而没法被其他Web实现的组件覆盖，有需要时应考虑隐藏。
* 因为是微信客户端代理网络通信，服务器返回的诸如set-cookie之类的headers是无法起作用的。
* WXSS中无法使用本地资源(例如，图片、字体等)，但可以引用网络资源。
* `<map/>`地图组件中`markers`标记点的`iconPath`属性无法使用网络资源，仅支持本地资源。
