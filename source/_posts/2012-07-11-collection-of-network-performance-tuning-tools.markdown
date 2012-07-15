---
layout: post
description: "寻找揭示linux网络及系统性能的工具 Collection of network performance tuning tools"
date: 2012-07-11 08:14
comments: true
categories: [linux,tool,network,performance]
keywords: "performance tuning, network, kernel, debug"
title: "寻找揭示linux网络及系统性能的工具"
---
##Performance Testing##
  * [Netperf](http://www.netperf.org/netperf/)
      - [Introduction](http://www.ibm.com/developerworks/cn/linux/l-netperf/)
  * [qperf](http://blog.yufeng.info/archives/2234)
  * [Tcpcopy](https://github.com/wangbin579/tcpcopy/)
      - [old version on googlecode](https://code.google.com/p/tcpcopy)
      - It is an online TCP duplication tool and can be used for testing (using netlink and raw
        sockets).And the author claimed it's better than [ab](http://httpd.apache.org/docs/2.0/programs/ab.html).
  * [Weighhttp](http://redmine.lighttpd.net/projects/weighttp/wiki)
  
##System Inspecting##
  * [PowerTop](https://www.linuxpowertop.org/powertop/)
  * [LatencyTop](https://latencytop.org)
      - [Introduction](http://rdc.taobao.com/blog/cs/?p=893)
  * [Systemtap](http://sourceware.org/systemtap/)
      - [Introduction](http://rdc.taobao.com/blog/cs/?p=477)
      - [Slides](http://www.slideshare.net/mryufeng/systemtap)
<!-- more -->
##Network Inspecting##
  * [tcprstat](http://www.percona.com/docs/wiki/tcprstat:start)
      - [Introduction](http://rdc.taobao.com/blog/cs/?p=728)

##Hardware Inspecting##
  * [cpu-topology](http://software.intel.com/en-us/articles/intel-64-architecture-processor-topology-enumeration/)
      - [Introduction](http://rdc.taobao.com/blog/cs/?p=460)
