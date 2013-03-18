---
layout: post
description: "check tcp connections"
date: 2012-08-09 11:39
comments: true
categories: [network,C,linux]
title: TCP状态迁移及状态码
keywords: linux, tcp, connection
---

{% imgpopup /images/post_img/Tcp_state_diagram.svg  50%  %}

想必大家对上图都比较熟悉了，补充下内核里对以上状态码的表示，顺便对源代码做了下改动，每个
状态的代码都补出来了。

{% codeblock tcp_state.h lang:c %}

enum {
	TCP_ESTABLISHED = 1,
	TCP_SYN_SENT    = 2,
	TCP_SYN_RECV    = 3,
	TCP_FIN_WAIT1   = 4,
	TCP_FIN_WAIT2   = 5,
	TCP_TIME_WAIT   = 6
	TCP_CLOSE       = 7, 
	TCP_CLOSE_WAIT  = 8,
	TCP_LAST_ACK    = 9,
	TCP_LISTEN      = 10,
	TCP_CLOSING     = 11,	/* Now a valid state */

	TCP_MAX_STATES	/* Leave at the end! */
};

{% endcodeblock   %}
<!-- more -->

用`cat /proc/net/tcp`打印时，输出所有active connections，
[http://search.cpan.org/~salva/Linux-Proc-Net-TCP-0.03/lib/Linux/Proc/Net/TCP.pm#The_/proc/net/tcp_documentation](输出格式介绍)。

`st`这一列即代表连接状态，下面该怎么做，你懂的。

要想快速高效的显示连接状态信息，推荐用(ss)[http://stackoverflow.com/questions/11763376/difference-between-netstat-and-ss-in-linux]。
Have fun.
