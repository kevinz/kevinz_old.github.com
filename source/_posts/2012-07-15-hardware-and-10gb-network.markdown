---
layout: post
description: "学习万兆网络 服务器平台 硬件 hardware and 10Gb network"
date: 2012-07-15 11:34
comments: true
categories: [network,linux,kernel,performance]
title: 学习万兆以太网，硬件和优化方法
keywords: 10GbE, linux, 优化
---
上周末去杭州参加了[ADC](http://adc.taobao.com/)，收获不小，关注的几个
topic都和万兆网有关。最近也在做用LVS替换F5的调研，对性能的要求非常高，
于是开始关注起10Gbe，记录一些资料和心得。

##个人关注点##
 * [拥抱万兆以太网时代 - 探索英特尔® 至强® 处理器平台上的应用性能优化](https://speakerdeck.com/u/gekben/p/by-intel)
   - 图片缩放及gzip的增强
   - E5-2600 DDIO这个绝对是重点，对网络收发性能提升明显
   - 性价比高的10G BASE-T会逐渐成为主流
 * [LVS在淘宝环境中的应用](https://speakerdeck.com/u/gekben/p/lvs-used-in-taobao)
   - FULLNAT模式，对大型网络降低运维难度非常好，但对LVS的改造很大，增加了复杂性
   - SynProxy，在lvs上实现syn cookie防御机制，不明白为什么要做在这里
   - taobao也没用上sandy bridge，还是E5640+intel 82599万兆卡
   - 多队列网卡，每个core绑定网卡的一个队列
   - KeepAlived增强，select->epoll,localaddr
   - Cluster，因为real server本身无状态，水平扩容好做，OSPF
   - 4/7层结合，lvs+nginx，nginx做7层业务负载
      
##硬件##
 * [介绍](http://storage.chinabyte.com/447/12275447.shtml)
   - [sandy bridge](http://en.wikipedia.org/wiki/Sandy_Bridge_%28microarchitecture%29) cpu列表
   - [x520](http://www.intel.com/content/www/us/en/network-adapters/gigabit-network-adapters/ethernet-x520.html)
   - [x540](http://ark.intel.com/products/58954/Intel-Ethernet-Converged-Network-Adapter-X540-T2)
   - [emulex 手头正好有这块卡](http://www.emulex.com/products/10gbe-network-adapters-nic/emulex-branded/oce11102-nt/overview.html)


##优化##
 * intel那个slides里有几个建议
   - Process affinity - socket
   - Interrupts affinity - socket
   - Disable LLDPAD,IPTABLES,IP6TABLES,SELINUX,IRQBALANCE
 * [Linux kernel and driver](http://timetobleed.com/useful-kernel-and-driver-performance-tweaks-for-your-linux-server/)
 * [GRO,GSO,TSO...](http://www.linuxfoundation.org/collaborate/workgroups/networking)
    
先记录到这里，边实践边补充。
      
      
