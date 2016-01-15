---
layout: post
title: Start Guide for BeagleBone Black
excerpt: "My notes to set up development environment on BBB"
modified: 2016-01-03
tags: [beaglebone, hardware]
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

{% highlight bash linenos %}
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

{% highlight bash linenos %}
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


