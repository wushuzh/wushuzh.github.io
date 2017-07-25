+++
date = "2017-06-26T22:31:22+08:00"
title = "遇到内核恐慌，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-kernel-panic/dead-comp.jpg"
draft = false
weight = 40
tags = [ "Howto", "rpm", "Linux" ]
+++

这个级别的报错，应该能撑起"咋整"系列的开篇门面。
<!--more-->


### 前言

《咋整》是在工作中，同学们遇到各式问题的案例汇总。之所以开一个系列写下来，一方面是因为有些问题很典型，今后还可能出现，一方面也是因为有些问题确实诡异，我也第一次见。一开始在公众号发出来时起的名是“活久见”系列。

### 现象分析

开篇的一个报错是——“内核恐慌”。

缘起是 Lucy 同学执行了一个貌似是查看系统负荷检测的脚本，但其中却包含了 rpm 的安装指令。没有然后，服务器直接挂掉：

- 现象1：当前已打开的会话，执行任何指令都会报错，relocation error
- 现象2：无法登入新用户，再重启系统后，直接挂起，直接显示 kernel panic

{{< highlight console >}}
/bin/login: relocation error: /usr/lib64/libc.so.6:
    symbol _ dl_starting_up, version GLIBC_PRIVATE not defined
    in file ld-linux-x86-64.so.2 with link time reference
{{< /highlight >}}
{{< figure src="/img/blog/deal-with-kernel-panic/panic-snap.png" title="Kernel Panic at Boot" >}}

### 处理过程

折腾一番，基本解决，简单记录一下：

1. 在 RedHat 知识库中找到了一个几乎一模一样报错信息，并且[解决方案](https://access.redhat.com/solutions/1475913)已被标记为验证通过；
2. 下载一个光盘镜像，通过服务器的远程管理，挂载ISO，并通过其引导系统；
3. 进入救援模式，查看环境，并对问题系统的关键目录文件截屏截图；
4. 执行第一条命令  chroot 时，不成功，依然报之前 relocation error 错误信息，看来此问题和知识库中找到的并不完全一样；
5. 跳过 chroot 直接到 /mnt/sysimage 下进行相关和新近文件进行各种移动及备份实验，截屏所有操作；
6. 错误信息最终转为找不到库函数文件，修复链接，重启后系统恢复

这里第五步稍微复杂些，有点像法医探伤，一般的方法就是阅读直接相关的脚本程序，查看相关 rpm 包中文件内容，和其他正常系统文件做比对，经过多轮猜测和尝试，最终你就一定能靠运气修好系统( 逃…… )

{{< figure src="/img/blog/deal-with-kernel-panic/centos-7-rescue-2.png" title="chroot" >}}

### 伴生知识

另外就是要了解一些不太常用的命令，知道去哪里查找就行：

* 查看一个 rpm 文件中的原数据 ```rpm -qip file.rpm```
* 查看一个 rpm 文件中的文件 ```rpm -qlpv file.rpm```
* 提取一个 rpm 文件中所有的文件 ```rpm2cpio file.rpm | cpio -idmv ```
* 查看 rpm 中预安装和后安装脚本内容 ```rpm -qp --scripts file.rpm ```
* 查看 rpm 中 spec 文件中关于安装过程的文件拷贝及权限设置等等，参考 Stackoverflow [extract the spec file from rpm package](https://stackoverflow.com/a/14168474)


{{< highlight bash >}}
// rpmrebuild is available in EPEL repo
rpmrebuild -s pkg.spec installed-pkg
rpmrebuild -e -p pkg.rpm
{{< /highlight >}}

### 事后总结  

最后是由表及里的经验总结：  

  1. 就算内核恐慌，自己也不用慌，尤其当你使用的是 RedHat 的时候，这通常意味着你使用的版本已落后了 8 - 10 年，所以你不会是第一个遇到此问题的倒霉蛋，利用好搜索引擎，操作时保留所有截屏及日志，方便之后问题回溯或上报。如果你有一个 RedHat 的订阅账户，优先在其知识库中查找，讨论，甚至要求技术支持
  2. 对开发软件的同学，尽量不要将工具的安装和执行混在一起，不得不这么做的时候，就要同时操作提示和环境检测，在业务需求、易用性、合规运维三方面做好平衡；
  3. 对拥有 sudo 权限或 root 密码的同学，一个重要的心理建设就是不能想当然，执行前要对来源不明的脚本做审核——记住蜘蛛侠他爷爷说的，能力越大，责任越大；

{{< figure src="/img/blog/deal-with-kernel-panic/spiderman.jpg" title="There must also come Great Responsibility" >}}


参考文档

> - Redhat Knowledge Base [How to boot a system into rescue mode](https://access.redhat.com/solutions/770703)
> - packagecloud blog (2015-10-13) [Inspecting and extracting RPM package contents]( https://blog.packagecloud.io/eng/2015/10/13/inspect-extract-contents-rpm-packages/)
> - [Fedora Packaging Guidelines for RPM Scriptlets](https://fedoraproject.org/wiki/Packaging:Scriptlets)

封面图片来自 [Dead Computer](https://dribbble.com/shots/538431-Dead-Computer) <a href="https://dribbble.com/doe_eyed"><i class="fa fa-dribbble" aria-hidden="true"></i> Eric Nyffeler</a>  

其他图片来源网络
