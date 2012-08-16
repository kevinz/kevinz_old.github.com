---
layout: post
title: "在SUSE Enterprise上安装Systemtap，解决build-id mismatch"
description: "install systemtap on suse"
date: 2012-08-16 20:49
comments: true
categories: [linux,tools,kernel]
keywords: linux, tools, kernel
---
最近做LVS的性能测试，自然会用的各种工具检视系统状态，用了很多工具，都
不甚满意，于是想尝试下Systemtap，系统环境是SUSE Enterprise SP2,已经3.0的
kernel，可以search到的suse上安装systemtap的内容不多，碰到问题并解决了
的更是少，自然我又要成为苦逼的问题解决者了。

##正常安装的步骤是##
 * 添加suse enterprise的包含debuginfo的iso，有几个G啊
 * `zypper in kernel-default-debuginfo`
 * `zypper in systemtap`
 
当我做完了以上这些，运行一个简单的脚本时:

{% codeblock systemtap hello_world lang:bash %}
sudo stap -ve 'probe begin { log("hello world") exit() }'
{% endcodeblock  %}
<!-- more -->

这个简单的例子是ok的，因为它不会涉及到systemtap的pass5，与kernel
module交互。跑下面这个的时候，就不行了:
{% codeblock systemtap hello_world lang:bash %}
stap -v -e 'probe vfs.read {printf("has VFS read()\n"); exit()}' 
{% endcodeblock  %}
会报出类似`ERROR: Build-id mismatch: "kernel" vs. "vmlinux" byte 0 (0xf9 vs 0xa2)`的问题，关于build-id mismatch这个问题，还是可以
搜索到不少资料的，不过告诉如何解决的也不多。先说明下这个问题，
`kernel-default-debuginfo`里面是附赠了一个kernel elf文件的，在SUSE里，
安装后放在`/usr/lib/debug/boot/vmlinux-3.0.13-0.27-default.debug`，这
是debug版本的kernel vmlinux文件，是elf原始文件，未经压缩。`stap`运行时会去检查它的。
执行`eu-readelf -n
/usr/lib/debug/boot/vmlinux-3.0.13-0.27-default.debug`，可以拿到的
build-id，执行结果的第一个字节是`f9`，那另外一个`a2`是从哪里得出来的，猜想应该和当
前运行的kernel有关，于是拿到当前运行的vmlinux的压缩文件，解压后检查
build-id：
{% codeblock check build-id lang:bash %}
   gunzip /boot/vmlinux-3.0.13-0.27-default.gz 
   eu-readelf -n /boot/vmlinux-3.0.13-0.27-default 
{% endcodeblock %}
执行结果也是`f9`，是对得上的啊。

但是注意，这份.gz文件是SUSE附赠的，跟当前跑的vmlinz文件没有关系，于是
开始打vmlinuz文件的主意，得先想办法把vmlinuz->vmlinux，及bzImage到elf，
才能去取它的build-id，
[https://www.globalways.net/blog/archives/76-Extracting-bzImage-from-vmlinuz.html](
这里)有可行的方法。
`eu-readelf -n vmlinux-unpacked`的结果果然是`a2`，我只能是怀疑SUSE给的
默认vmlinuz有问题了，不能和SUSE提供的用于debuginfo的vmlinux匹配。

只能另外想办法，其实systemtap做kernel mismatch判断的关键代码，就在:

{% codeblock systemtap/runtime/sym.c lang:c %}
static int _stp_build_id_check (struct _stp_module *m,
            unsigned long notes_addr,
            struct task_struct *tsk)
{
...
/* just hack it */
if (rc || (theory != practice)) {
      const char *basename;
      basename = strrchr(m->path, '/');
      if (basename)
|   basename++;
      else
|   basename = m->path;

#if LINUX_VERSION_CODE >= KERNEL_VERSION(2,6,27)
      _stp_error ("Build-id mismatch: \"%s\" vs. \"%s\" byte %d (0x%02x vs 0x%02x) address %#lx rc %d\n",
|   |     m->name, basename, j, theory, practice, notes_addr, rc);
      return 1;
#else
      /* This branch is a surrogate for kernels affected by Fedora bug
       * #465873. */
      _stp_warn (KERN_WARNING
|   |    "Build-id mismatch: \"%s\" vs. \"%s\" byte %d (0x%02x vs 0x%02x) rc %d\n",
|   |    m->name, basename, j, theory, practice, rc);
#endif
      break;
    } /* end mismatch */
    
{% endcodeblock %}

关键就在于`if (rc || (theory != practice))`，小小修改的一下，写成:
`if (0 && (rc || (theory != practice)))`就OK了，绕过了检查。然后重新编
译systemtap，直接用的SUSE提供的老版本的源码，新的会有问题，注意还要用
到SUSE提供的`libdwfl`的source code，才能完成编译，编译时在
`/usr/src/packages/SOURCES/elfutils-0.152/libdwfl/linux-kernel-modules.c`
文件上遇到了问题，稍作修改就可过关。尝试下新编译出来的`stap`，完全OK，它是不知
道'build-id mismatch`为何物的，敢这样改也就是因为都是offical的东西，本
应该match得上，应该不会出什么大问题，实践中检验吧。

Have fun.
