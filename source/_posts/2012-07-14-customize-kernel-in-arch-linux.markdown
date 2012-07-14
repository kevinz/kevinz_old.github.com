---
layout: post
title: "customize-kernel-in-arch-linux"
date: 2012-07-14 16:54
comments: true
categories: [linux]
keywords: "arch linux, kernel, compile"
description: "在archlinux里定制编译内核"
---
心血来潮升级到
[linux-ck_3.4.4-4](http://repo-ck.com/x86_64/linux-ck-corex-3.4.4-4-x86_64.pkg.tar.xz)
后，发现`pm-suspend`出问题了，可以`suspend`，但`resume`后显示屏亮不起
来，其它部分都`resume`成功了，可以隐约看见一些窗口的黑影。于是我又开始
折腾了，参考了
[这篇](https://bbs.archlinux.org/viewtopic.php?id=143545)，插句题外话，
不知道不是巧合，好多问题都是在archlinux的相关论坛上找到了答案，都是一帮爱折腾的同好。

不幸的是，[这篇](https://bbs.archlinux.org/viewtopic.php?id=143545)把
我引导向了错误的方向，决定把kernel降级到3.3.x。因为我用的不是官核，不
好找old archive，必须重新编译3.3.x-ck的内核，官方的old archives可以在
[这里](http://arm.konnichi.com/search/)寻觅到。

拜[Arch Build System](https://wiki.archlinux.org/index.php/Arch_Build_System)
所赐，编译、打包、patch的步骤都可以内置了，对于kernel也是一样。贴一部
分PKGBUILD文件的内容，是从
[repo-ck的archive](http://repo-ck.com/PKG_source/linux-ck/linux-ck-3.3.8-1.src.tar.gz)
里找到的。
<!-- more -->
{% codeblock PKGBUILD %}
### PATCH AND BUILD OPTIONS
_makenconfig="n"	# tweak kernel options prior to a build via nconfig
_localmodcfg="n"	# compile ONLY probed modules
_use_current="n"	# use the current kernel's .config file
_BFQ_enable_="n"	# enable BFQ as the default I/O scheduler

pkgname=linux-ck
true && pkgname=(linux-ck linux-ck-headers)
_kernelname=-ck
_basekernel=3.3
pkgver=${_basekernel}.8
pkgrel=1
arch=('i686' 'x86_64')
url="https://wiki.archlinux.org/index.php/Linux-ck"
license=('GPL2')
options=('!strip')
_ckpatchversion=1
_ckpatchname="patch-${_basekernel}-ck${_ckpatchversion}"
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/3.3.0-v3r3"
source=("http://www.kernel.org/pub/linux/kernel/v3.x/linux-3.3.tar.xz"
"http://www.kernel.org/pub/linux/kernel/v3.x/patch-${pkgver}.xz"
"http://ck.kolivas.org/patches/3.0/3.3/${_basekernel}-ck${_ckpatchversion}/${_ckpatchname}.bz2"
'linux-ck.preset'
'config' 'config.x86_64'
'change-default-console-loglevel.patch'
'i915-fix-ghost-tv-output.patch'
'ext4-options.patch'
'fix-acerhdf-1810T-bios.patch'
"${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-v3r3-3.3.patch"
"${_bfqpath}/0002-block-introduce-the-BFQ-v3r3-I-O-sched-for-3.3.patch")
sha256sums=('355df2085626cdf0083c4bc0fe3017419034b6db5cce6f437ae8234a5e90b40c'
            '4bbd173c995f44cb1bc5b002293765e8d8f9f076ae801f35cf456da9f0eba06d'
            'dd23a8e0fc6cdb393a9659d5d097c27e1ff5a0f30d0fe6a3441512a9bf6d00cb'
            'c2cf8cc2600502de348f3dc3aae9a3bde5486759db15cb8a93df7aa35bd6e7da'
            '253b11d36850a9c06ad8861a339da8895ddbb875d4e61579447a26f13423a160'
            '0f6def8badb9939ddb042657fd767c3a07ea7c291ed795935ebc8f81255231d7'
            'b9d79ca33b0b51ff4f6976b7cd6dbb0b624ebf4fbf440222217f8ffc50445de4'
            '9ccadbe3eb30bb283af3eb869c3a4bdb764628854811cc616a2e02e9ef398705'
            '0f15e7462b5d2650c354580920978228b3092bcf47e20e600242c8ca102df6f5'
            'f77e1a1f2d955f0499dc2957d1d65a3eb931212a13948b33916332296d0e4a7a'
            '179e159deaaa3976f5e5e329a640050b9803fef3cc77f8588a949b28c732c5ad'
            'a401732b5bc36eeccbaab86b216295d0ce0b30b8afdae66d8f59a382689da6a4')
build() {
	cd "${srcdir}/linux-${_basekernel}"

	msg "Patching with upstream patch ${_basekernel} to ${_basekernel}.4"
	# add upstream patch
	#patch -p1 -i "${srcdir}/patch-${pkgver}"

	# add latest fixes from stable queue, if needed
	#http://git.kernel.org/?p=linux/kernel/git/stable/stable-queue.git

	### Patch with BFQ IO Scheduler
	msg "Patching source with BFQ patches"
	#for p in $(ls ${srcdir}/000{1,2}-block*.patch); do
	#	patch -Np1 -i $p
	#done

	### Clean tree and copy ARCH config over
	msg "Running make mrproper to clean source tree"
	make mrproper

	if [ "${CARCH}" = "x86_64" ]; then
		cat "${srcdir}/config.x86_64" > ./.config
	else
		cat "${srcdir}/config" > ./.config
	fi

	### Optionally use running kernel's config
	# code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
	if [ $_use_current = "y" ]; then
		if [[ -s /proc/config.gz ]]; then
			msg "Extracting config from /proc/config.gz..."
			# modprobe configs
			zcat /proc/config.gz > ./.config
		else
			warning "You kernel was not compiled with IKCONFIG_PROC!"
			warning "You cannot read the current config!"
			warning "Aborting!"
			exit
		fi
	fi

	if [ "${_kernelname}" != "" ]; then
		sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
	fi

	### BFQ to be compiled in but not enabled
	sed -i -e s'/CONFIG_CFQ_GROUP_IOSCHED=y/CONFIG_CFQ_GROUP_IOSCHED=y\nCONFIG_IOSCHED_BFQ=y\nCONFIG_CGROUP_BFQIO=y/' \
		-i -e s'/CONFIG_DEFAULT_CFQ=y/CONFIG_DEFAULT_CFQ=y\n# CONFIG_DEFAULT_BFQ is not set/' ./.config

	### Optionally enable BFQ as the default io scheduler
	if [ $_BFQ_enable_ = "y" ]; then
		sed -i -e '/CONFIG_DEFAULT_IOSCHED/ s,cfq,bfq,' \
			-i -e s'/CONFIG_DEFAULT_CFQ=y/# CONFIG_DEFAULT_CFQ is not set\nCONFIG_DEFAULT_BFQ=y/' ./.config
	fi

	# set extraversion to pkgrel
	sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

	# get kernel version
	msg "Running make prepare for you to enable patched options of your choosing"
	make prepare

	### Optionally load needed modules for the make localmodconfig
	# See http://aur.archlinux.org/packages.php?ID=41689
	if [ $_localmodcfg = "y" ]; then
		msg "If you have modprobe_db installed, running it in recall mode now"
		if [ -e /usr/bin/modprobed_db ]; then
			[[ ! -x /usr/bin/sudo ]] && echo "Cannot call modprobe with sudo.  Install via pacman -S sudo and configure to work with this user." && exit 1
			sudo /usr/bin/modprobed_db recall
		fi
		msg "Running Steven Rostedt's make localmodconfig now"
		make localmodconfig
	fi

	if [ $_makenconfig = "y" ]; then
		msg "Running make nconfig"
		make nconfig
	fi

	msg "Running make bzImage and modules"
	make ${MAKEFLAGS} bzImage modules
}

package_linux-ck() {
...
{% endcodeblock %}

比较长，但也很易读懂，验证资源文件，解压资源文件，打patch，获取config，最后`make
bzImage modules`，可以在`export MAKEFLAGS="-j4"`，充分利用多核cpu加快
编译。读上面的配置文件可以知道，对于32/64的系统，已经提供了默认的
config文件，可以直接修改config，或者以`make menuconfig`的方式，这里有
个小技巧：
{% codeblock 获取依赖的资源并解压 lang:bash %}
$tar -xzvf linux-ck-3.3.8-1.src.tar.gz
$cd linux-ck
$makepkg -s
{% endcodeblock %}
ABS会自动按照PKGBUILD的步骤开始执行，可以在编译真正开始时，终止其执行，
可以在linux-ck目录下，发现src目录，所有patch，kernel源代码都放在这里了，
进入kernel源码目录，`make menuconfig`就可以对默认的config进行定制修改
了，具体步骤就不赘述了，大家慢慢体会。

用这种方法编译出来的kernel，是以`.xz` package的形式存在，就跟在
`/var/cache/pacman/pkg/`下的官方package一样，可以很好管理、跟踪。

最后解决休眠恢复的问题，是降级成nvidia-295.53-2的驱动解决的，可以用在
最新的3.4.4-4-ck内核上，确实是驱动的问题，期间换用nouveau的开源驱动也
有同样问题。
