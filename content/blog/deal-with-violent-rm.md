+++
date = "2017-06-29T21:38:48+08:00"
title = "删错了，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-violent-rm/pc-gone.png"
draft = false
weight = 41
tags = [ "Howto", "rpm", "Linux" ]
+++

蛮力卸载后，啥都不能用了……
<!--more-->

## 问题背景

Han 同学在虚拟机上做补丁安装测试时，蛮力将 nss 卸载后，IP尚能 ping 通，但 ssh、sftp 都不能用了，求救。

通过 MDN 快速了解一下这个核心库：
基本上所有和因特网安全的标准协议都离不开这个库，平时大家常用的一些集成、依赖此库的产品有 Firefox , Thunderbird , Pidgin , 开源Office , RedHat Apache SSL module , Oracle Java System 旗下的各种产品。

{{< highlight bash >}}
rpm -qi nss # get metadata of a installed pkg
{{< /highlight >}}
<br />

## 问题重现

陷落的是一个 KVM 虚拟机，直接登录到其宿主服务器上：``` virsh list ```查到依然是运行状态。既然不能 ssh 登录，通过``` virsh console guestname ```登录其管理端口，做所有后续的维护操作。

问题重现倒是很容易，随便执行一条ssh命令，系统日志 /var/log/message 里也充斥着类似的报错信息：

{{< highlight console >}}
sshd: /usr/sbin/sshd: error while loading shared libraries:
    libssl3.so: cannot open shared object file:
        No such file or directory
{{< /highlight >}}
<br />

## 查找原因

先查看 yum 的包更新历史，没有和肇事时间相符的记录。

{{< highlight bash >}}
yum history [info someid]
yum history package-list nss
{{< /highlight >}}

再查看 history 命令记录，找到了这条
{{< highlight bash >}}
rpm -e --nodeps nss
{{< /highlight >}}

这个 nodeps 参数导致跳过了 rpm 删除前的依赖性检测，是一个典型的暴力型删除。
之后的命令历史就是试图补救：尝试用 scp 从外面拷贝一个包进来，但没了 nss 根本无法建立和外部服务器的链接。

## 修复过程

先试着用 yum 修复：因为系统已经配置上了处在公司内网的 rpm 安装源，并通过 http 传输文件，并不需要 SSL 等安全链接。但试过才知道这也不行，同样报之前找不到 libnss3.so 错误。如此看来 yum 内部集成了更多安全校验。其首字母 Y 代表 Yellowdog 确实是超出预期的靠谱。

之后我又试着下载了删除的安装包，直接拷到出事机器上：先将虚拟机关机，再在宿主服务器上执行 virt-copy-in 这样就不用依赖 ssh sftp scp 的安全传输链路了。但后面用 rpm 安装的时候，还是报同样的错误信息。其实也不惊讶，yum 只是封装了 rpm 的一个前端管理工具。

{{< highlight bash >}}
yum install --downloadonly --downloaddir=dir pkgname
// 或 yumdownloader pkgname
virt-copy-in -d guestvm srcfile-on-host targt-vm-path
{{< /highlight >}}

最终可行的办法有[两个](https://www.centos.org/forums/viewtopic.php?t=18457#p87783)：

1. 找一个一模一样架构的工作正常的机器，仅仅把缺失的文件直接拷过来，一旦 yum/rpm 可用不再报错，立刻通过本地 rpm 文件或 yum 源执行包安装，完善修复
2. 找不到一样的机器，就挂上安装介质，进入救援模式，进而继续安装

{{< highlight bash >}}
sudo virt-ls -l -d guestvm /usr/lib64/ |wc -l
sudo virt-ls -l -d guestvm /usr/lib64/libnss3.so
sudo virt-copy-in -d guestvm /tmp/libnss3.so /usr/lib64/
virsh console guestvm # to exec yum install nss
rpm --root /mnt/sysimage nss
{{< /highlight >}}
<br />

## 经验总结

* 除非明确知道自己在做什么，尽量不要用类似 nodeps 这样的参数。
> 不加上这个危险参数，这种错误就不会发生，无论 rpm 还是 yum 一定会在动手前做出检测，为你列出被执行对象现有的所有依赖。当你看到 100 行以上各式已安装在你系统内的软件依赖信息时，你也就不会真的下狠手将其删除了。

> 通过 repoquery 命令查询一个这个给定 PKG_NAME 包被多少已经安装在当前系统中的包依赖（意味着不能轻易删除）

{{< highlight bash >}}
repoquery --whatrequires --installed --recursive PKG_NAME
{{< /highlight >}}

> 这里注意不要和 ```yum deplist PKG_NAME``` 弄混，后者是列出给定包依赖哪些包提前安装好


* 有 yum 就不要用 rpm : 规范的做法是设置一个本地 yum 源做各种包的更新，这会让服务器管理员，运维同事的日子好过很多。具体的好处有：
> - 历史操作查找审计，一旦有问题，可回溯当时操作的详情
> - 通过事务型的回滚，到任意之前的老版本，包括相应的依赖库的回滚
> - 非用 rpm 的话，一定要区分 install 和 upgrade，用 -i 参数会导致系统中存在统一软件的多个版本，如果是更新包，应该使用升级而非安装的参数

{{< highlight bash >}}
yum history
yum downgrade specific_pkg_ver
rpm -U --oldpackage [file.rpm]
{{< /highlight >}}
<br />

* 重要操作之前要做虚拟机快照
> 物理机也就算了，虚拟机打快照已经非常方便了。S/L大发好，老司机都知道。

{{< highlight bash >}}
virsh snapshot-create-as [dom] [snapname] <desc>
{{< /highlight >}}
<br />

缩略语解释

> - MDN: Mozilla Developer Network 开发者中心，维护了不少 Web 标准和 Mozzila 产品的文档，个人感觉比 w3schools 专业靠谱很多，另外 Alexa 排名也比后者考前不少 (128 vs 185)
> - S/L大法: Save / Load(存储 / 读取）的简称，在某些涉及乱数、随机数、独立概率的游戏中，通过不断的存档、读档以达到玩家预期的结果。

参考文档

> - MDN   [Overview of NSS Open Source Crypto Libraries]( https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Overview
>)
> - RHEL6 [Working with Transaction History]( https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sec-Yum-Transaction_History.html)
> - Aaron K (2017-3-15) [How to Use ‘Yum History’ to Find Out Installed or Removed Packages Info](https://www.tecmint.com/view-yum-history-to-find-packages-info/)
> - RedHat Knowledge Base [How to use yum to download a package without installing it](https://access.redhat.com/solutions/10154)
> - RedHat Knowledge Base [How to use yum to downgrade or rollback some package updates?](https://access.redhat.com/solutions/29617)
> - James Olin Oden (2004-05-01) [Transactions and Rollback with RPM](http://www.linuxjournal.com/article/7034)
> - [How do I downgrade an RPM?](https://serverfault.com/a/274311)
> - Fedora forum nucleo comments (2011-10-28) [List package dependents (not dependencies)](http://forums.fedoraforum.org/showthread.php?t=271492)

文中图片来源网络

封面图片来自 [I'm a computa](https://dribbble.com/shots/1606433-I-m-a-computa) <a href="https://dribbble.com/Tymn"><i class="fa fa-dribbble" aria-hidden="true"></i> Tymn Armstrong</a>  
