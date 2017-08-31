+++
date = "2017-08-07T19:18:15+08:00"
title = "需要外援，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-ks-failure2/q-encil.png"
draft = false
weight = 46
+++

在搞不定而向他人求援时，该做些什么？
<!--more-->

## 网上搜索

首先利用好搜索引擎，寻找类似甚至一模一样的问题的已有解决方案。
搜索范围包括但不限于 Google 搜索第一页所有链接，特别注意 StackExchange 旗下热门子站，如 Stack Overflow、Server Fault、Super User，Redhat Knowledgebase， HP Support forum ……

比如 Redhat 知识库中标记为已解决的帖子

> - [Installation failed with "storage configuration failed: Unable to allocate requested partition scheme." in RHEL 7 with multipath device](https://access.redhat.com/solutions/1233263)
> - [Installation failed with "storage configuration failed: Unable to allocate requested partition scheme." in RHEL7](https://access.redhat.com/solutions/1258933)

其中解决方案是 a. 通过 uuid 制定磁盘、b. 显式重置磁盘元数据 ```clearpart --all --initlabel ```，这和我当下 ks 文件的做法一致。

另外，从 [rhinstaller anaconda releases](https://github.com/rhinstaller/anaconda/releases) 上看，我使用的版本也并非最新版，我猜这个问题也存于安装程序的底层的块设备配置模块 [storaged-project/blivet](https://github.com/storaged-project/blivet)。

## 求援外部

RedHat 大部分产品开源，其商业模式主要来自对其产品的商业支持。你如果时它的付费订阅会员，是可以向其咨询问题，并索要技术支持的。RedHat 作为产品源码的拥有/维护者、或方案整合方，其视野和对技术理解的深度可以极大加速问题处理的进程。而对个人来说，这是一个非常重要的学习机会。

提问前最好先通读用户手册，一个像 anaconda 这样高成熟度的软件(存在并被广泛数年)的多数问题都源于用户对工具错误理解和不当配置或使用。

总之，应正确理解产品特性，同时确认过相同或类似问题的解决方案对当下情形无效，并自己尝试过所有能力所及的调试后，再咨询外部专家才是解决问题的最高效路径。

然后，就是通过其问题上报系统，撰写问题描述，交互讨论问题。这部分做的好坏对问题处理的进度影响很大。比如，

- 能否将自己的业务逻辑很好剥离，简单直接的将问题的核心症状表述清楚？
- 能否正确执行外部专家给你的调查步骤，并将结果做高效反馈？

> 而我见到的对上报 bug 最好的建议来自 Simon Tatham (Putty作者) [How to Report Bugs Effectively](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html) 这个文章有[中文翻译](https://www.chiark.greenend.org.uk/~sgtatham/bugs-cn.html)，建议打印一份放在贴在工位上，随时反复阅读。

> 在更为宽泛的定下，对于如何提问的最好建议来自 Eric Steven Raymond 和 Rick Moen 的 [How To Ask Questions The Smart Way](http://www.catb.org/esr/faqs/smart-questions.html)，也有中文翻译：[ryanhanwu](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way) 和 [ZRONG](http://doc.zengrong.net/smart-questions/cn.html)

这么大张旗鼓的引用了这些如何提问、提 bug 的建议，还是觉得这是技术工作者一个重要的基本素质。从结果看，凡是提问前做充分思考的同学，不但自身技术提高得快，同时也能得到其所在组织和同事的认可、肯定。

## 日志收集

有的同学遇到 OS 安装错误，习惯于直接拿出手机对着出错屏幕拍摄，但其实上篇 “[安装失败，咋整？]({{< relref "blog/deal-with-ks-failure1.md" >}})” 中已经提到用户可以通过进入各个预设的虚拟终端做相应的日志查看和命令执行的。

比如当你撰写问题描述时不可避免的要收集服务器错误发生时已经产生的所有日志，按照 [What log files should I gather to troubleshoot a kickstart or manual installation issue with Anaconda?](https://access.redhat.com/solutions/20358) 的要求，你需要收集已有日志

- /tmp/anaconda.log
- /tmp/ifcfg.log
- /tmp/program.log
- /tmp/storage.log
- /tmp/syslog
- /tmp/yum.log

执行指令产生更多的日志并打包:

{{< highlight console >}}
# dmesg &> /tmp/dmesg.log
# fdisk -l &> /tmp/fdisk.log
# parted -sl &> /tmp/parted.log
# cat /proc/partitions &> /tmp/proc_part.log
# multipath -ll &> /tmp/mpath.log
# ls -lR /dev &> /tmp/dev.log
# cat /proc/scsi/scsi &> /tmp/proc_scsi.log
# tar cvf installation_logs.tar /tmp/*log*
{{< /highlight >}}

<br />

## 日志回传

除了上述在问题服务器 A 上逐条执行命令，最后打包的方式，还可以在其他服务器 B 上准备好一个 shell 脚本，包含所有需要执行的指令。这样只要服务器 A 配好网络，就可以将此脚本从服务器 B 上拷贝回来，执行生成日志后，在拷出到服务器 B 上。

{{< highlight bash >}}
#!/bin/bash
# prepare this shell script named LogCapture on server B
cd /tmp; mkdir cmd-outputs
W() { local OutputFile=$(tr ' /' '_' <<<"$@"); $@ >cmd-outputs/$OutputFile; }
[[ ! -f ks.cfg ]] && echo No /tmp/ks.cfg present >ks.cfg
ls anaconda-tb-* &>/dev/null || kill -USR2 $(</var/run/anaconda.pid)
W cat /mnt/sysimage/root/anaconda-ks.cfg
W date
W dmesg
W dmidecode
W lspci -vvnn
W fdisk -cul
W ls -lR /dev
W dmsetup ls --tree
W lvm pvs
W lvm vgs
W lvm lvs
W cat /proc/partitions
W mount
W df -h
W cat /proc/meminfo
W cat /proc/cpuinfo
W ps axf
W lsof
W ip -s li
W ip a
W ip r
date=$(date +%F)
tar cvjf install-logs-$date.tbz *log cmd-outputs anaconda-tb-* ks.cfg
echo -e "\nFinished gathering data\nUpload /tmp/install-logs-$date.tbz to another system\n"
{{< /highlight >}}

查看当前你的服务器 A 是否已经获得了有效的网络地址，如果没有，你需要通过 dhcp 或是指令配置好服务器的 ip addr。然后通过 scp、rsync 将之前生成的日志回传。

{{< highlight console >}}
server A # dhclient -v eth0
server A # ifconfig eth0 xx.xx.xx.xx/24 up
server A # rsync -v User@Remote:LogCapture /tmp
server A # bash /tmp/LogCapture
server A # rsync -v /tmp/install-logs-*.tbz User@Remote:
{{< /highlight >}}

<br />


> 关于网络设置，参见  
> - [10 Useful “IP” Commands to Configure Network Interfaces](https://www.tecmint.com/ip-command-examples/)  
> - [How to Change Your IP Address From the Command Line in Linux](https://www.howtogeek.com/118337/stupid-geek-tricks-change-your-ip-address-from-the-command-line-in-linux/)  
> - [How To Execute Ping Command Only For N number of Packets](http://www.thegeekstuff.com/2009/11/how-to-execute-ping-command-only-for-n-number-of-packets/)

另外，我发现 anaconda-installer 的文档中有更为高阶的操作，可以在服务器 A 上[启动一个 sshd 服务](https://anaconda-installer.readthedocs.io/en/latest/boot-options.html#inst-sshd)，这样只要网络调通，我们就可以远程连入做各种调试了。

## 专家分析

最后答案揭晓，经过两位 Redhat 工程师的支持，最终发现了一块磁盘的大小设置过大的错误——一块 300 GB 的硬盘实际能使用的大小只有 286 GB。在 ```storage.log``` 中相关日志如下：

{{< highlight console >}}
# ================= storage.log =================

DEBUG blivet: allocating partition: raid.18 ; id: 617 ;
  disks: [u'sdo'] ;
  boot: False ; primary: False ; size: 314572800000 ;
  grow: False ; max_size: 0 B ; start: None ; end: None
DEBUG blivet: checking freespace on sdo
DEBUG blivet: getBestFreeSpaceRegion: disk=/dev/sdo
  part_type=0 req_size=314572800000 boot=False
  best=None grow=False start=None
DEBUG blivet: checking 63-585871963 (286070)
DEBUG blivet: current free range is 63-585871963 (286070)
DEBUG blivet:
  not enough free space for primary -- trying logical

{{< /highlight >}}

这几行报错信息离最终爆出的磁盘不够的错误信息都在 storage.log 中，但之间相差了大概 2000 行日志，这也就是这个问题一开始我没能准确定位错误的原因。遇到存储相关的问题还是要细读 storage 日志，并考虑用下面的指令过滤出相关行做逐一分析是这次分析获得宝贵经验。

{{< highlight console >}}
# grep -e "allocatePartitions" -e "allocating partition" \
  -e "checking freespace" -e "new free" \
  -e "current free range is" -e max_size \
  storage.log
{{< /highlight >}}

### 参考文档

> - [Anaconda src intro](https://fedoraproject.org/wiki/Anaconda/SourceOverview) 对 anaconda 安装程序内部模块的简介
> - [Fedora Blivet](https://fedoraproject.org/wiki/Blivet) 为啥选 Blivet aka 魔鬼叉子 这么彪悍的名字
> - [Community-Kickstarts](https://github.com/CentOS/Community-Kickstarts) 有一些可供参考的 ks 配置文件
> - [AUTOMATIC INSTALLATION with Kickstart file](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-simple-install-kickstart.html) 介绍了如何将一个 ks 文件加入安装介质的步骤

封面图片来自 [?encil](https://dribbble.com/shots/461456--encil) <a href="https://dribbble.com/kyotux"><i class="fa fa-dribbble" aria-hidden="true"></i> Asher</a>
