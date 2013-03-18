---
layout: post
description: "Setup uml network development environment"
title: "搭建基于User Mode Linux网络模块开发环境"
date: 2012-11-13 15:40
comments: true
categories: [linux,kernel,network]
keywords: "linux, uml, user mode linux,vde"
---
最近在写一些内核网络模块，需要一个好用的编译、调试、测试环境，选用了uml，记录一下设置多台uml和host通信，并share fs。
uml基本安装和使用可以参考[archlinux uml](https://wiki.archlinux.org/index.php/User-mode_Linux#Setup_by_rootfs_.2B_tap)。
{% codeblock start uml with vde support lang:bash %}
# 先启动虚拟switch
#vde_switch -s /tmp/switch1 -tap tap0 -m 666
# 启动一台单网卡的uml，已经与之前定义的switch绑定
#vmlinux ubd0=arch_rootfs1  mem=256M eth0=vde,/tmp/switch1
# 启动一台双网卡的uml，已经与之前定义的switch绑定
#vmlinux ubd0=arch_rootfs2  mem=256M eth0=vde,/tmp/switch1 eth1=vde,/tmp/switch1
{% endcodeblock %}

<!-- more -->

启动完成后，在linux里用常规的网络设置方法完成ip地址的分配。
{% codeblock network configuration lang:bash %}
#ifconfig eth0 192.168.0.101 up
#ifconfig eth1 192.168.0.101 up
#确定host上的tap0已经正确设置并启用
#ifconfig tap0 192.168.0.100 up
{% endcodeblock %}
完成后用ping测试下，各个uml和host之间的网络通信是否正常。

接下来设置uml与主机的文件系统共享。
{% codeblock mounting hostfs directory lang:bash %}
#mkdir -p /root/source
#mount none /root/source/ -t hostfs -o /home/yourname/code/
{% endcodeblock %}

完成后，应该可以读写hostfs的文件，测试一下，放一份linux源代码在host的`/home/yourname/code/`下，
目录结构是`/home/yourname/code/linux`，自己写的module的目录在`/home/yourname/code/my_module`，
对应在uml上的目录就是`/root/source/linux`和`/root/source/my_module`。
{% codeblock Makefile of my_module :bash %}
obj-m := test.o #source file is test.c
KDIR := /root/source/linux
PWD := $(shell pwd)
default:
        $(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules
{% endcodeblock %}

在`/root/source/my_module`下`make`就可以了，在`#insmod test.ko`测试，如果crash了就`kill`掉host上的`vmlinux`进程，非常方便快速。
{% codeblock kill all vmlinux process lang:bash %}
#killall vmlinux processes if uml can't be started
#killall vmlinux
{% endcodeblock %}

当然，gdb也是可以用的，可以参考uml gdb调试相关的介绍。




