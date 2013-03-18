---
layout: post
title: 基于lvs的全透明传输思考 
description: "full transparent with lvs"
date: 2012-11-13 17:28
comments: true
categories: [linux,lvs,kernel,network]
keywords: "lvs, transparent, firewall" 
---
互联网公司一般使用LB的方式，是把Web App Server作为real server，最简单的情况是请求在这里终结，响应由Web App Server返回，一般web server不用再访问外网了。再复杂的情况，就
是Web App Server又作为client去请求内网的其它资源，如DB，其它Web Server等，在这种典型的互联网部署模式下，一般可以把Web Server看作终点。

还有一种部署方式，load balancing的对象不是Web App Server，而是firewall系统，比如用linux搭建的firewall，经过firewall的流量需要全透明传输，即不修改layer 2以上的信息，firewall
对过滤后合格的包做forwarding。在这种部署模式下，LB也需要有全透明传输的能力，画个图解释一下:

{% imgpopup /images/post_img/lvs_trans_firewall.png 70%  %}
<!-- more -->

这里所说的LB都是lvs，以http为例，首先client请求WWW Server，client有自己的公网地址，组装好的ip header，source ip是cip，destination ip是www_ip，
source port是cport，destination port 80，lvs要支持这种destination ip非vip的部署，需要使用到fwmark。
首先是要mark出这个请求：
{% codeblock mark the traffic lang:bash %}
#iptables -t mangle -A PREROUTING -p tcp --dport 80 -j MARK --set-xmark 1
{% endcodeblock %}

然后交由lvs处理：
{% codeblock direct traffic to local_in hook lang:bash %}
#ip rule add prio 101 fwmark 1 table 101
#ip route add local 0/0 dev lo table 10
{% endcodeblock %}

lvs的fwmark service及real server：
{% codeblock define lvs fwmark service and real servers hook:bash %}
#ipvsadm -A -f 1 -p #default scheduler is wlc,enable persistent
#ipvsadm -a -f 1 -r rs1 #default xmit method is direct routing 
#ipvsadm -a -f 1 -r rs2
#ipvsadm -a -f 1 -r rs2
#ipvsadm -a -f 1 -r rs2
{% endcodeblock %}

这样配置后，LB1充当了透明的load balancer，经过LB1的request的layer2以上的信息都没有变化，rs1~rs4都可以接收请求，
并进行过滤处理，也是全透明的。先考虑没有LB2的情况，request从rs出来，被转发到Router，Router在将request转发至外网，
直至WWW_Server。从Client到WWW_server，layer2以上的地址和端口信息都没有改变。

Response从WWW_Server出发，source是www_ip：80，destination是cip:cport，发向Router，Router这时候不知道该
如何进行路由了，因为从rs1出来的request，对应的response也要回到rs1，Router没有rs1的内部ip，也没有连接信息
参考，response回不到对应real server，连接无法建立起来。

产生这个问题的原因是real server是有状态的，在request经过它时做了记录，例如linux的防火墙的基础conn_track，它期待response
也必须经过这台real server，tcp/udp以及基于它们的应用层协议都是如此。为了让response能回到正确的real server，需要利用到一种技术，
所有request和response都要经过它转发，在request进来时，记录它的源mac地址和接收包的device，response回来时，绕过路由直接发送给
之前记录的源mac，维护连接表+mac地址是必须的。

目前支持这种技术的设备有F5，被称为Auto Last Hop，还有Juniper的某些设备，称作Reverse Route。把这种设备部署在LB2的位置，即可实现
full transparent，当然，也不是一定要单独搞一台LB2，用一台LB既做Load Balancer又做反向路由也是可以的。使用这种设备的最大缺点就是，
贵。于是想基于Lvs实现方向路由功能，想想也是非常合理的，lvs本身可以很好的维护管理连接状态，它有一张connection table，连接状态迁移，
超时机制，垃圾回收都已经非常成熟，再加上source mac和in_dev并不难。需要注意的是，lvs在这种使用模式下，应该定义fwmark service，它的rs只有一个，
就是下一跳的ip。经过一段时间的开发，对lvs进行修改后，基于lvs的反向路由功能已经OK，准备下一篇来讲讲细节。




