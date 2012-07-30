---
layout: post
description: "Study on LVS kernel code - persistent"
date: 2012-07-11 12:24
comments: true
categories: [linux,kernel,network,lvs,C]
title: "LVS peristent代码分析"
keywords: "LVS, ipvs, persistent, code, implementation"
---
一直纠结于LVS使用persistent时，是以何为依据，决定一个新的连接请求的命运的：是被persistent管理，去到之前的real server，还是被调度算法重新调度，去到新的real server。
代码里其实写得非常清楚：
##Persistent in LVS(ipvs)##
 * fwmark
   - <IPPROTO_IP,caddr,0,fwmark,0,daddr,0> 这个六元组，最开始是写死的
     ip协议，第一个0是cport，第二个0是dport，就是不在乎cport和dport，
     这个daddr值得一提，经过debug发现，这个值为`0.0.0.fwmark`。
 * Port zero service <protocol,caddr,0,vaddr,0,daddr,0>
 * non Port zero service
   - FTP <caddr,0,vaddr,0,daddr,0>
   - NON-FTP <caddr,0,vaddr,vport,daddr,dport>
<!-- more -->    
如果是transparent mode，这种透明模式一般都是通过fwmark的方式实现，客户
端是不知道vip的存在的，比如用`iptables`设置了`fwmark`为3，则访问
http://sina.com.cn时，`daddr`就是`0.0.0.3`。

见代码17，21，77，80行。
{% codeblock /net/netfilter/ipvs/ip_vs_core.c lang:c %}
    /*
	 * As far as we know, FTP is a very complicated network protocol, and
	 * it uses control connection and data connections. For active FTP,
	 * FTP server initialize data connection to the client, its source port
	 * is often 20. For passive FTP, FTP server tells the clients the port
	 * that it passively listens to,  and the client issues the data
	 * connection. In the tunneling or direct routing mode, the load
	 * balancer is on the client-to-server half of connection, the port
	 * number is unknown to the load balancer. So, a conn template like
	 * <caddr, 0, vaddr, 0, daddr, 0> is created for persistent FTP
	 * service, and a template like <caddr, 0, vaddr, vport, daddr, dport>
	 * is created for other persistent services.
	 */
	if (ports[1] == svc->port) {
		/* Check if a template already exists */
		if (svc->port != FTPPORT)
			ct = ip_vs_ct_in_get(svc->af, iph.protocol, &snet, 0,
					     &iph.daddr, ports[1]); /* <caddr,0,vaddr,vport,daddr,dport> */
		else
			ct = ip_vs_ct_in_get(svc->af, iph.protocol, &snet, 0,
					     &iph.daddr, 0);        /* <caddr,0,vaddr,0,daddr,0> */

		if (!ct || !ip_vs_check_template(ct)) {
			/*
			 * No template found or the dest of the connection
			 * template is not available.
			 */
			dest = svc->scheduler->schedule(svc, skb);
			if (dest == NULL) {
				IP_VS_DBG(1, "p-schedule: no dest found.\n");
				return NULL;
			}

			/*
			 * Create a template like <protocol,caddr,0,
			 * vaddr,vport,daddr,dport> for non-ftp service,
			 * and <protocol,caddr,0,vaddr,0,daddr,0>
			 * for ftp service.
			 */
			if (svc->port != FTPPORT)
				ct = ip_vs_conn_new(svc->af, iph.protocol,
						    &snet, 0,
						    &iph.daddr,
						    ports[1],
						    &dest->addr, dest->port,
						    IP_VS_CONN_F_TEMPLATE,
						    dest);
			else
				ct = ip_vs_conn_new(svc->af, iph.protocol,
						    &snet, 0,
						    &iph.daddr, 0,
						    &dest->addr, 0,
						    IP_VS_CONN_F_TEMPLATE,
						    dest);
			if (ct == NULL)
				return NULL;

			ct->timeout = svc->timeout;
		} else {
			/* set destination with the found template */
			dest = ct->dest;
		}
		dport = dest->port;
	} else {
		/*
		 * Note: persistent fwmark-based services and persistent
		 * port zero service are handled here.
		 * fwmark template: <IPPROTO_IP,caddr,0,fwmark,0,daddr,0>
		 * port zero template: <protocol,caddr,0,vaddr,0,daddr,0>
		 */
		if (svc->fwmark) {
			union nf_inet_addr fwmark = {
				.ip = htonl(svc->fwmark)
			};

			ct = ip_vs_ct_in_get(svc->af, IPPROTO_IP, &snet, 0,
					     &fwmark, 0); /* <IPPROTO_IP,caddr,0,fwmark,0,daddr,0> */
		} else
			ct = ip_vs_ct_in_get(svc->af, iph.protocol, &snet, 0,
					     &iph.daddr, 0); /* <protocol,caddr,0,vaddr,0,daddr,0> */

		if (!ct || !ip_vs_check_template(ct)) {
			/*
			 * If it is not persistent port zero, return NULL,
			 * otherwise create a connection template.
			 */
			if (svc->port)
				return NULL;

			dest = svc->scheduler->schedule(svc, skb);
			if (dest == NULL) {
				IP_VS_DBG(1, "p-schedule: no dest found.\n");
				return NULL;
			}

			/*
			 * Create a template according to the service
			 */
			if (svc->fwmark) {
				union nf_inet_addr fwmark = {
					.ip = htonl(svc->fwmark)
				};

				ct = ip_vs_conn_new(svc->af, IPPROTO_IP,
						    &snet, 0,
						    &fwmark, 0,
						    &dest->addr, 0,
						    IP_VS_CONN_F_TEMPLATE,
						    dest);
			} else
				ct = ip_vs_conn_new(svc->af, iph.protocol,
						    &snet, 0,
						    &iph.daddr, 0,
						    &dest->addr, 0,
						    IP_VS_CONN_F_TEMPLATE,
						    dest);
			if (ct == NULL)
				return NULL;

			ct->timeout = svc->timeout;
		} else {
			/* set destination with the found template */
			dest = ct->dest;
		}
		dport = ports[1];
	}
{% endcodeblock %}
