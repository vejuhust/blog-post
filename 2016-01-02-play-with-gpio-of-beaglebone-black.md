---
layout: post
title: Play with GPIO of BeagleBone Black
excerpt: "My incomplete guide to GPIO programming on BBB"
modified: 2016-01-02
tags: [coding, programming, beaglebone, hardware]
comments: true
---

{% include _toc.html %}


# Setup

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
