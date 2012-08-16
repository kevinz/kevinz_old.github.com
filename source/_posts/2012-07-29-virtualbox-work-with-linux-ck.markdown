---
layout: post
description: "how virtualbox work with linux-ck"
date: 2012-07-29 18:29
comments: true
categories: [linux tools arch]
title: how virtualbox work with linux-ck 
keywords: linux-ck, virtualbox
---
解决了virtualbox在linux-ck里使用的问题，做一下记录。我用的是
linux-ck-corex内核，直接通过
[repo-ck](https://wiki.archlinux.org/index.php/Repo-ck)装好，不过也可
以通过[aur](http://aur.archlinux.org/packages.php?ID=50911)来折腾下，
调整下内核参数啥的。这里主要记录下如何让virtualbox正常工作，[这篇](https://wiki.archlinux.org/index.php/Linux-ck#Running_Virtualbox_with_Linux-ck)提到了
一些相关内容，做一点补充:
<!-- more -->

##编译vboxdrv及其它模块##
 * 记得装对应的linux-ck-headers，我装的是linux-ck-corex-headers，可能
   要先fuck gfw。
 * 编译方法有所变化，老版的virtualbox是用`vboxbuild`，新版改成了`dkms
   install vboxhost/4.1.18`。
 * 解决编译vboxdrv报错问题。
   - headers安装目录默认是在`/usr/src/kernel_full_name`，在
     `/lib/modules/kernel_full_name`下，执行`ln -s
     /usr/src/kernel_full_name build`，编译应该就ok了。
 * 老版本的virtualbox编译好的modules是存在于`/lib/modules/extramodules`
   下，新的是在`/lib/modules/kernel/misc`下。
 * add/append your user into vboxusers group
 
Have fun.
