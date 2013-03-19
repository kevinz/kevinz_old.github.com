---
layout: post
title: "Kernel dynamic debugging and taints"
date: 2013-03-19 09:41
comments: true
categories: [linux,kernel,debug]
description: "how to apply kernel dynamic debugging" 
keywords: "config_dynamic_debug, kernel debug, kernel printk" 
---
最近在为自己写的LKM添加调试功能时，发现了一个比printk更好的选择，就是[dynamic debugging](http://lwn.net/Articles/434833/)。

{% codeblock list all dynamic_debug enabled items lang:bash %}
#mkdir /debug
#mount -t debugfs debugfs /debug
// 看下哪些地方使用了dynamic_debug机制
#cat /debug/dynamic_debug/control
#echo ' +p' > /sys/kernel/debug/dynamic_debug/control
{% endcodeblock %}
这样就能打开所有的dynmaic_debug items，对于Linux kernel source tree里的代码，dynamic_debug是适用的，但自己写的LKM似乎用不了这个机制，于是花了些时间想一探究竟。

来看下pr_debug或者dev_dbg的定义:
{% codeblock /include/linux/printk.h lang:c %}
/* If you are writing a driver, please use dev_dbg instead */
#if defined(DEBUG)
#define pr_debug(fmt, ...) \
        printk(KERN_DEBUG pr_fmt(fmt), ##__VA_ARGS__)
#elif defined(CONFIG_DYNAMIC_DEBUG)
/* dynamic_pr_debug() uses pr_fmt() internally so we don't need it here */
#define pr_debug(fmt, ...) \
        dynamic_pr_debug(fmt, ##__VA_ARGS__)
#else
#define pr_debug(fmt, ...) \
        no_printk(KERN_DEBUG pr_fmt(fmt), ##__VA_ARGS__)
#end
{% endcodeblock %}
<!-- more -->

可见`CONFIG_DYNAMIC_DEBUG`是必须开的,`zcat /proc/config.gz|grep -i dynamic_debug`可查。

做了调试后发现，关键是在加载LKM时被识别为对kernel有taint的模块。
{% codeblock /include/linux/printk.h lang:c %}
err = check_module_license_and_versions(mod);
...
// 只对安全的LKM开启dynamic_debug
if (!mod->taints || mod->taints == (1U<<TAINT_CRAP)){
    dynamic_debug_setup(info.debug, info.num_debug);
{% endcodeblock %}

写了个最简单的LKM，license是GPL，加载时还是有问题，不知道是不是必须编译进Kernel才可以，等做了实验再更新一下。


