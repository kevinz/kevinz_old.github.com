---
layout: post
title: "升级Netxen网卡驱动及firmware"
date: 2012-08-16 20:15
comments: true
description: "upgrade the netxen nic"
categories: [linux,network]
keywords: linux, network, nic, driver
---
在公司做lvs测试用机是HP DL580 G7，除了网卡，其它的配置都不错，网卡是板
载的netxen的1G卡，还有几张netxen的10G卡，观察`/proc/interrupts`，每块
10G卡对应的中断都只有4个，而且没有识别成tx-0/rx-0的形式，不像支持
multiple queue，不禁使我对SUSE 11
Enterprise SP2的netxen驱动产生的怀疑。`dmesg`也可以发现有抱怨driver和
firmware的比较旧，于是萌生了升级驱动的想法。
<!-- more -->

用`netxen`和`nx3031`search
了很久而不得，偶然通过`nx_nic`找到了HP网卡的驱动
[http://h20000.www2.hp.com/bizsupport/TechSupport/SoftwareIndex.jsp?lang=en&cc=us&prodNameId=3913538&prodTypeId=329290&prodSeriesId=3913537&swLang=8&taskId=135&swEnvOID=4049](
下载地址)，包括firmware的自动升级程序和网卡驱动。值得吐一把嘈的是，HP
网站上driver是老旧的，不支持稍新的内核，貌似只支持到2.6.20以前的，编译
时会报`net_dev struct`相关的错，费了
那么大的力气，搞来个不能用的，之后还是google到了最新的驱动，在一个连域
名都没有的
[ftp://15.217.49.75/ftp2/pub/softlib2/software1/pubsw-linux/p280489831/](ftp)
上，haha，真欢乐，HP sucks.按照说明先升级好driver，`modprobe nx_nic`后，
就可以升级firmware了，要注意的是，把每个网卡都up起来，才能保证所有都升
级成功，我拆掉了一些bond，保证以前是slave状态的卡都up起来之后，经过10
分钟左右，升级成功。重启后`dmesg`里的complains少了很多，新版本也显示出
来了，还是没识别出multiple queue，编译驱动时也检查了选项，确实没有。同
事也咨询过HP的sales，也不知道多队列网卡为何物，与Intel的资料的专业性相比，我对
HP的网卡实在是无力吐嘈。之后研究下再驱动代码吧，有代码有真相。

Have fun.
