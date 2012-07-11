---
layout: post
title: "Netfilter hello world module"
date: 2012-07-07 23:16
comments: true
categories: [linux]
description: "hello world之netfilter模块"
keywords: "linux, kernel, netfilter"
---
对于linux网络的学习，学下写netfilter module更有利于理解，下面开始实战。
{% gist 3066821 %}
<!-- more -->
{% gist 3066825 %}

然后就是
{% codeblock install the module lang:bash %}
#make
#insmod hello.ko 
{% endcodeblock %}

我的环境(archlinux64 3.31-ck)编译加载成功，但貌似会导致kernel crash，
哈哈，后面再解决吧，have fun。
