---
layout: post
title: "了解lvs调试"
date: 2012-07-26 21:32
comments: true
categories: [linux,network,kernel]
description: "lvs debug howto"
keywords: linux, kernel, ip_vs, lvs, network
---
虽然对lvs的实现代码也算是心里有数了，但遇到一些具体问题时还拿不准，这时候就想到了lvs提供的调试功能，
是内核提供的一个选项。一般发行版默认是没打开的，至少我接触的suse enterprice和archlinux都是关闭
了ip_vs_debug，一起来看看怎么启用吧。
{% codeblock kernel config  %}
CONFIG_IP_VS_DEBUG=y
{% endcodeblock %}

在默认的config文件里找到这一项，可能是注释状态，改好后，就可以开始编译内核了，这个选项不是m，必须重新
编译产生新内核，在用`mkinitcpio -k kernel_full_name -c config`或者`mkinitrd -k kernel_full_name
`，前面是针对archlinux的，后面是针对suse的，这里有坑，此处略去1000字，反正记得如果用新内核启动，出现
找不到root的情况，八成是initrd做得有问题，fs相关的modules的问题。

新内核一切安好后，就可以玩lvs的调试功能了。先运行下`ipvsadm`，再`lsmod`看下`ip_vs`是否已经加载了。
再后面，就可以检查期待已久的调试选项了
{% codeblock ip_vs debug in /proc %}
/proc/sys/net/ipv4/vs/debug_level
{% endcodeblock %}

`cat debug_level`一下，默认值是0，以后可以通过sysctl来更改默认值，下面讲下这个值的含义:
{% codeblock ip_vs_core.c lang:c %}
 IP_VS_DBG_BUF(1, "Forward ICMP: failed checksum from %s!\n", IP_VS_DBG_ADDR(af, snet));
{% endcodeblock %}
上面是一个debug macro的调用的例子，下面是宏的具体定义：

{% codeblock ip_vs.h lang:c %}
#ifdef CONFIG_IP_VS_DEBUG
extern int ip_vs_get_debug_level(void);
#define IP_VS_DBG(level, msg...)			\
    do {						\
	    if (level <= ip_vs_get_debug_level())	\
		    printk(KERN_DEBUG "IPVS: " msg);	\
    } while (0)
#define IP_VS_DBG_RL(msg...)				\
    do {						\
	    if (net_ratelimit())			\
		    printk(KERN_DEBUG "IPVS: " msg);	\
    } while (0)
#define IP_VS_DBG_PKT(level, pp, skb, ofs, msg)		\
    do {						\
	    if (level <= ip_vs_get_debug_level())	\
		pp->debug_packet(pp, skb, ofs, msg);	\
    } while (0)
#define IP_VS_DBG_RL_PKT(level, pp, skb, ofs, msg)	\
    do {						\
	    if (level <= ip_vs_get_debug_level() &&	\
		net_ratelimit())			\
		pp->debug_packet(pp, skb, ofs, msg);	\
    } while (0)
#else	/* NO DEBUGGING at ALL */
#define IP_VS_DBG(level, msg...)  do {} while (0)
#define IP_VS_DBG_RL(msg...)  do {} while (0)
#define IP_VS_DBG_PKT(level, pp, skb, ofs, msg)		do {} while (0)
#define IP_VS_DBG_RL_PKT(level, pp, skb, ofs, msg)	do {} while (0)
#endif
{% endcodeblock %}
可以清晰的看到，打印debug日志的条件是`level <= ip_vs_get_debug_level()`，这个函数调用的返回值就是
`/proc/sys/net/ipv4/vs/debug_level`，万物皆文件的理念又体现了。
可见level的值越小，其优先级越高，查了一遍ip_vs的相关代码，level最大值不过12，也就是说，
只要把`/proc/sys/net/ipv4/vs/debug_level`设置成12，就能保证所有ip_vs的日志输出了，但一般没那个必要。
{% codeblock set debug_level to 12 %}
echo 12 > /proc/sys/net/ipv4/vs/debug_level
{% endcodeblock %}
对了，日志输出是在dmesg。

好了，方法就差不多是这些了，接下来就边看代码，边测试，边看输出吧。
have fun.
