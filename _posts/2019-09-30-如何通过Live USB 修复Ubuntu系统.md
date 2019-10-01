---
layout: post
post: true
title: 通过Live USB修复损坏的Ubuntu系统
date: 2019-09-30
category: Ubuntu
tag: 故障修复
author: Feng Yuan
---

* content
{: toc}


### 硬盘上的Ubuntu系统出现了问题的修复方式

使用Linux的时候，经常会出现，系统故障，比如：误删系统文件，内核版本不匹配，误删重要驱动等问题，导致Linux系统无法进入。遇到这种情况，终极手段是通过USB上的Linux登录，拷贝出重要资料之后，重新装系统。这里介绍一种不需要重新装系统的修复原系统的方法。

**1.使用U盘制作一个Linux启动盘并登录**

**2.使用如下程序mount原系统到U盘系统下**

{%highlight bash%}
sudo mount / /mnt
sudo mount -o bind /proc /mnt/proc
sudo mount -o bind /dev /mnt/dev
sudo mount -o bind /dev/pts /mnt/dev/pts
sudo mount -o bind /sys /mnt/sys
sudo cp -L /etc/resolv.conf /mnt/etc/resolv.conf
sudo chroot /mnt /bin/bash
{%endhighlight%}

&nbsp;
其中，倒数第二行 `sudo cp -L /etc/resolv.conf /mnt/etc/resolv.conf`, 用于让原系统联网。这样就可以在U盘系统下使用原系统中还没有损坏的一切工具来修复原系统了。

**3.可以使用原系统中的一切工具修复原系统**
这个时候，就可以使用apt等工具，在损坏了的硬盘系统上安装或者重新安装驱动等损坏或者误删的程序。