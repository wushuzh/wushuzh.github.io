+++
date = "2017-08-01T19:12:33+08:00"
title = "安装失败，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-ks-failure1/kickstart-car.png"
topImage = "/img/blog/deal-with-ks-failure1/kickstart-car.gif"
draft = false
weight = 45
+++

如何调查 RedHat 上失败的 Kickstart 安装？
<!--more-->

## 问题上报

之前 “[启动错误，咋整？]({{< relref "blog/deal-with-boot-failure.md" >}})”里提到的服务器终于要被重装，之前听说服务器的重大变动，如重启，重装，升级，搬移都最好先看黄历、拜雍正(很冷的**八阿哥**的梗)，因为太容易出现意想不到的问题。

而这台很长时间不曾重装过的机器在安装时果然遇到了问题。所报问题如下：

{{< highlight console >}}
Starting installer, one monent...
anaconda 21.40.22.93-1 for Red Hat Enterprise Linux 7.3 started.
* installation iog files are stored in /tmp during the installation
* shell is available on TTY2
* when reporting a bug add logs from /tmp
      as separate text/plain attachments
91:51:09 Running pre-installation scripts
99:56:44 Not asking for VNC because of an automated install
99:56:44 Not asking for VNC because text mode was explicitly asked
             for in kickstart
99:56:44 Not asking for VNC because we don't have a network
Starting automated install............
Checking software selection
Generating updated storage configuration
storage configuration failed:
    Unable to allocate requested partition scheme.
=================================================================
=================================================================
Installation

1) [x] Language settings         2) [x] Time settings
       (English (United States))        (Asia/Shanghai timezone)
3) Lx] Installation source       4) [x] Software selection
       (Local media)                    (Custom software selected)
5) [!] Installation Destination  6) [x] Xdump
       (No disks selected)              (Kdump is enabled)
7) [ ] Network configuration     8) [ ] User creation
       (Not connected)                  (No user will be created)
Not enough space in file systems for the current software selection.
    An additional 1793.04 NiB is needed.
Enter 'b' to ignore the warning and attempt to install anyway.
Please make your choice from above
['q' to quit | 'b' to begin installation | 'r' to refresh]:


[anaconda] 1:main* 2:shell 3:log 4:storage-log 5:program-log   
                                   Switch tab: Alt+Tab | Help: F1

{{< /highlight >}}

<br />

## 问题分析

安装的虽然是最常见的 RHEL，但我们也为产品做了安装中各种选项的深度优化和定制。

为让部署的简单高效，我们借助 kickstart 将安装细节——尤其是分区挂接点的规划和磁盘冗余设置——封装起来。最终只呈现一个仅用来收集安装最基础信息的极简界面给最终用户。

上面看到的报错信息来自 [anancoda](https://github.com/rhinstaller/anaconda) 安装程序中的 TUI 安装界面 [python-simpleline](https://github.com/rhinstaller/python-simpleline)，隐约记得早先的文字安装界面就是直接使用 tmux (查看文档最新的 rhel7.4 仍然是[这样](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-consoles-logs-during-installation-x86.html))。应该是后来 Redhat 工程团队为了让用户更易用或更便捷定制的目的，新开发这个 simpleline 。无论如何，tmux 和simpleline 效果上都创建了多个终端来分别记录不同类别的日志。比如上面报错就出现在了 1 号名为 **main** 的子页面。

而当出现报错时，用户可以按照界面的提示，按下 ```Alt + Tab``` (simpleline) 或 ```Ctrl + B then TTY# ```(tmux) 亦或 ```Ctrl + Alt + F<n> ```跳转到不同终端，查看各方面的日志，也可进入 shell 环境中做一些基本的调试和日志打包收集工作。

- 1 号：安装主窗口
- 2 号：进入一个拥有 root 权限的可以执行有限命令的 shell
- 3 号：显示安装日志，其内容也存储在 /tmp/anaconda.log
- 4 号：显示存储日志，其内容也存储在 /tmp/storage.log
- 5 号：显示其他日志，其内容也存储在 /tmp/program.log

比如4号虚拟终端中，存储相关的报错信息为：

{{< highlight console >}}
INFO anaconda: Parsing kickstart : /run/install/ks.cfg
DEBUG anaconda: scripts found for locale en_US.UTF-8: ['Latn']
DEBUG anaconda: console fonts found for locale en_US.UTF-8: ['lat...
DEBUG anaconda: setting console font to latarcyrheb-sunl6
DEBUG anaconda: console font set successfullj to latarcyrheb-sunl6
DEBUG anaconda: setting locale to: en_US.UTF-8
DEBUG anaconda: network: devices found ['eth0', 'eth1', 'eth...
DEBUG anaconda: network: ifcfg file for eth0 not found
DEBUG anaconda: network: pre kickstart - adding connection for eth0
DEBUG anaconda: network: kickstart pre section applied for dev ...
DEBUG anaconda: network: ifcfg file for ethl not found
DEBUG anaconda: network: ifcfg file for eth2 not found
DEBUG anaconda: network: ifcfg file for eth3 not found
DEBUG anaconda: network: missing ifcfgs created for devices ['eth1'...
DEBUG anaconda: network: setting ONBOOT value of eth0 to True
DEBUG anaconda: network: real kickstart ONBOOT value set for dev ...
INFO anaconda: Running Thread: Anat4aitForConnectingNMThread ...
WARN anaconda.stdout: Not asking for VNC because of an automated ...
WARN anaconda.stdout: Not asking for VNC because text mode was ...
WARN anaconda.stdout: Not asking for VNC because we don't have ...
INFO anaconda: Thread Done: AnaWaitForConnectingNMThread ...
INFO anaconda: Display mode = t
INFO anaconda: 264241152 kB (258040 MB) are available
INFO anaconda: check_memory(): total:250040, needed:320, graphical:410
INFO anaconda: Red Hat Enterprise Linux is the highest priority, ...
INFO anaconda: boot loader GRUB2 on X86 platform
INFO anaconda: boot loader GRUB2 on X86 platform
INFO anaconda: Running Thread: AnaStorageThread (139679552460736)
INFO anaconda: Running Thread: AnaTimeInitThread (139679504647936)
INFO anaconda: Running Thread: AnaPayloadRestartThread ...
INFO anaconda: Running Thread: AnaPayloadThread (139679187862528)
INFO anaconda: Thread Done: AnaPayloadRestartThread (139679496255232)
INFO anaconda: Thread Done: AnaTimeInitThread (139679504647936)
INFO anaconda: Running Thread: AnaSourceWatcher (139679584647936)
INFO anaconda: Running Thread: AnaStorageWatcher (139679496255232)
INFO anaconda: Thread Done: AnaStorageThread (139679552468736)
INFO anaconda: Thread Done: AnaStorageWatcher (139679496255232)
INFO anaconda: Thread Done: AnaPayloadThread (139679487862528)
INFO anaconda: Thread Done: AnaSourceWatcher (139679504647936)
DEBUG anaconda: new disk order: [u'sda', u'sdb']
DEBUG anaconda: Boot loader: use 'sda' first disk from driveorder
                    as boot drive, dry run True
ERR anaconda: storage configuration failed:
                  Unable to allocate requested partition scheme.
INFO anaconda: fs space: 0 B needed: 1793.84 MiB
ERR anaconda: Not enough space in file systems
    for the current software selection.
        An additional 1793.84 NiB is needed
INFO anaconda: Running Thread: AnaInputThread1 (139679504647936)



[anaconda] 1:main 2:shell- 3:log* 4:storaqe-log 5:program-log
                                     Switch tab: Alt+Tab | Help: F1
{{< /highlight >}}

对于界面上产生的错误，你同样也可以做屏幕截图 ```Shift + Print Screen```。相关的问题会被保存在 ```/tmp/anaconda-screenshots/``` 目录下


## 深入调查

此系统内部2块 SATA 盘，6块 SSD 固态硬盘，并通过 4 个 P812 磁盘阵列控制器，外接了 4个 MSA 2040 存储，总计 48 块 SATA 盘。这么多外接磁盘，设计时需要兼顾为系统提供较高的可靠性和读写性能，即 18 个逻辑分区采用硬件 RAID ，在其之上再用软 RAID 进行组合。

<img alt="ext lds" src="/img/blog/deal-with-ks-failure1/ext-lds.png" class="img-responsive">

进入 2 号虚拟终端 执行```lsscsi -s```确认系统实际逻辑卷和和上述期待相符。关于 SG 更多使用说明，详见 Doug Gilbert (2016-7-4) [The Linux SCSI Generic (sg) Driver](http://sg.danny.cz/sg/index.html) 相关 lsscsi 的页面。

从日志上看，错误发生的阶段为，ks 安装程序 anaconda 在估算需要安装的 rpm 软件包后，发现没有足够空间？ 但这部分的 partition 规划是在服务器内置的 2 块 SATA 盘上。

而且从命令的输出来看，这两块盘总大小 300 GB，根分配了 50 GB，就算把所有包都选中，做 10 个完整安装也足够了。但不知为什么抛出了现在这个错误。查看 ks 中对内外挂硬盘的 partition 定义——其内容达有 32 条指令之多。这些指令在指向相应磁盘时也都使用了块设备的唯一标识 uuid——感觉也很稳妥。

关于 uuid 和 /dev 下对应关系，可以通过```blkid```及```lsblk```查看，参见Adrian Dinu (2014-11-23) [Linux blkid Command to Find Block Devices Details](https://linoxide.com/linux-command/linux-command-lsblk-blkid/)

到这，我准备找外援来帮忙查看这个问题了。下一篇我会详细复述一下这个求助的过程。

封面图片来自 [Kickstart Car](https://dribbble.com/shots/2342802-Kickstart-Car) <a href="https://dribbble.com/KristianDuffy"><i class="fa fa-dribbble" aria-hidden="true"></i> Kristian Duffy</a>  
