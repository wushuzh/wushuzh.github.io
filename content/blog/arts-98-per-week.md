+++
date = "2020-04-27T23:58:01+08:00"
title = "ARTS 2020w17"
image = "/img/blog/arts-98-per-week/vr-dashboard.webp"
draft = true
weight = 2017
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

[#1290. Convert Binary Number in a Linked List to Integer](https://leetcode.com/problems/convert-binary-number-in-a-linked-list-to-integer/) 题目要求：已知一个二进制的数字，通过一个单链表的数据结构来表示，要求实现一个算法，将其换算为对应的十进制数字。

单链表读取顺序是高位先读，低位后读，所以考虑先获得二进制数的字符串形式，然后通过 int 转化为 10 进制数字。

{{< highlight python >}}
def get_decimal_value(head):
    dec_str = ''
    cur = head
    while cur:
        dec_str += str(cur.val)
        cur = cur.next
    
    return int(dec_str, 2)
{{< /highlight >}}

一些可有可无的优化——不再转化为字符串——通过位移操作符("<<")，不断升位，而且题设没要求不能变更给定链表，直接通过它迭代循环，省一个变量 `cur` 的创建。

{{< highlight python >}}
def get_decimal_value(head):
    dec = 0
    while head:
        dec <<= 1
        dec += head.val
        head = head.next

    return dec
{{< /highlight>}}

## Review

## Tip

## Share

#### 利用 Teamviewer 通过服务器上的 LTE 模块远程运维

联想新出了一款“边缘”服务器 [SE350](https://www.lenovo.com/us/en/data-center/servers/edge/ThinkSystem-SE350/p/77XX6DSSE35) ，比如突出的特点有：

- 物理尺寸上：宽度和深度和一般的笔记本电脑差不多，厚度 4 厘米；
- 内置 LTE 模块，插上一个手机SIM卡后，可实现互联网访问；
- 内置一个路由器——安装 openWRT 系统：可以根据使用场景完成对无线网口（LTE，WIFI）、有线网口（万兆/千兆/BMC口）的互联互通配置；

<br/>

有了 LTE 模块，**远程运维/控制**就成为了可能，但通过 LTE 获得的 IP 可能存在 NAT，无法直连，而潜在的解决方案有

a. 服务器上安装启动 Teamviewer/向日葵 ，远程 PC 上也安装 TV/向日葵，即可通过 LTE 网络实现点对点控制服务器  
b. 在 openWRT 上配置 openVPN ，为远程 PC 客户端创建证书，一样通过 LTE 网络，创建 VPN 连接到服务器，类似这里的介绍：[How to Setup Multiple OpenVPN Server to Different VLANs](https://openwrt.org/docs/guide-user/services/vpn/openvpn/multiple-vpns-to-vlans)  
c. 通过公网服务器，通过反向 ssh 实现访问  
d. 同样利用公网服务器，安装配置 [frp](https://github.com/fatedier/frp/blob/master/README_zh.md)  

关于 a 方案 (Teamviewer)，搭建简单，但是安全性，稳定性其实不可控，遇到问题自己调试起来也是非常麻烦。

TV 自己的知识库中有几篇的文档，但信息组织还是显得很凌乱，看完后不能得到有用信息：

- [How to install TeamViewer on Linux without graphical user interface](https://community.teamviewer.com/t5/Knowledge-Base/How-to-install-TeamViewer-on-Linux-without-graphical-user/ta-p/4352#)
- [How do I use TeamViewer on headless systems?](https://community.teamviewer.com/t5/Knowledge-Base/How-do-I-use-TeamViewer-on-headless-systems/ta-p/4256)
- [Which ports are used by TeamViewer?](https://community.teamviewer.com/t5/Knowledge-Base/Which-ports-are-used-by-TeamViewer/ta-p/4139#)
- [Which operating systems are supported?](https://community.teamviewer.com/t5/Knowledge-Base/Which-operating-systems-are-supported/ta-p/24141)
- [How to install TeamViewer for Linux](https://community.teamviewer.com/t5/Knowledge-Base/How-to-install-TeamViewer-for-Linux/ta-p/6318)
- [How to install TeamViewer on Red Hat and CentOS](https://community.teamviewer.com/t5/Knowledge-Base/How-to-install-TeamViewer-on-Red-Hat-and-CentOS/ta-p/30708)


记录一下我尝试的步骤：

1. 申请一个手机 SIM 卡（和主套餐绑定的副卡），打开服务器，找到 LTE 模块，插入 SIM 卡；
2. SE350 安装 centos ，我安装 了 GUI
3. SSH 连接到 SE350 的 embedded switch 上，将 LTE 网络定向到操作系统内，测试一下通过 LTE 的互联网连接，比如做一下 yum 系统更新
4. 在 centos 下安装配置 Teamviewer/向日葵
5. 防火墙配置

第四步的时候，启动 TV 时，出现报错: 

{{< highlight txt >}}
2020/05 ... S   SecureNetworkConnection::SendCallbackHandler(): [ remoteID: 42 connection: 1 remoteConnection: 0 ], Error: RCommand (Timeout)
2020/05 ... S   SecureNetworkConnection::SendCallbackHandler(): [ remoteID: 42 connection: 1 remoteConnection: 0 ] Resetting connection due to error RCommand (Timeout)
2020/05 ... S   AnonymousConnection::ConnectAndRun(): SecureNetwork connection failed state:0
2020/05 ... S!! Communication::Messenger::SendAnonymousMessage: error: Could not connect, Errorcode=11
{{< /highlight >}}

从 TV 的用户论坛发现了这篇帖子[TM is able to connect but refuses to login (connection problems blabla) (ubuntu)](https://community.teamviewer.com/t5/Linux/TM-is-able-to-connect-but-refuses-to-login-connection-problems/td-p/42912)

然后我将我的 teamviewer 降级到 14.3.4730-0 ，然后忽然就可以连接了，猜测可能是这个版本在 linux 下支持良好，不过重启服务器后，启动 TV，发现又连不上，各种修复没用，然后忽然就又能连上了，于是又猜测是网络的问题。

无论如何，复习一下 yum 的操作：

- 使用 yum 安装一组软件 `yum grouplist hidden`, `yum groupinstall "<group name>"`
- 使用 yum 查看某个软件的所有可用版本 `yum --showduplicates list teamviewer-host`；
- 使用 yum 安装特定版本的完整包名的拼接方法：[package-name]-[version].[architecture]
- 使用 yum 回退软件 [How to use yum to downgrade or rollback some package updates?](https://access.redhat.com/solutions/29617)

{{< highlight txt >}}
# yum install epel-release

# cat teamviewer.repo

[teamviewer]
name=TeamViewer - $basearch
baseurl=http://linux.teamviewer.com/yum/stable/main/binary-$basearch/
gpgkey=http://linux.teamviewer.com/pubkey/currentkey.asc
gpgcheck=1
enabled=1
type=rpm-md
failovermethod=priority

# yum repolist all

Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.bfsu.edu.cn
 * epel: hkg.mirror.rackspace.com
 * extras: mirrors.ustc.edu.cn
 * updates: mirrors.ustc.edu.cn
repo id                                   repo name                                                              status
C7.0.1406-base/x86_64                     CentOS-7.0.1406 - Base                                                 disabled
C7.0.1406-centosplus/x86_64               CentOS-7.0.1406 - CentOSPlus                                           disabled
C7.0.1406-extras/x86_64                   CentOS-7.0.1406 - Extras                                               disabled
C7.0.1406-fasttrack/x86_64                CentOS-7.0.1406 - Fasttrack                                            disabled
C7.0.1406-updates/x86_64                  CentOS-7.0.1406 - Updates                                              disabled
......
C7.7.1908-base/x86_64                     CentOS-7.7.1908 - Base                                                 disabled
C7.7.1908-centosplus/x86_64               CentOS-7.7.1908 - CentOSPlus                                           disabled
C7.7.1908-extras/x86_64                   CentOS-7.7.1908 - Extras                                               disabled
C7.7.1908-fasttrack/x86_64                CentOS-7.7.1908 - Fasttrack                                            disabled
C7.7.1908-updates/x86_64                  CentOS-7.7.1908 - Updates                                              disabled
base/7/x86_64                             CentOS-7 - Base                                                        enabled: 10,070
base-debuginfo/x86_64                     CentOS-7 - Debuginfo                                                   disabled
base-source/7                             CentOS-7 - Base Sources                                                disabled
c7-media                                  CentOS-7 - Media                                                       disabled
centos-kernel/7/x86_64                    CentOS LTS Kernels for x86_64                                          disabled
centos-kernel-experimental/7/x86_64       CentOS Experimental Kernels for x86_64                                 disabled
centosplus/7/x86_64                       CentOS-7 - Plus                                                        disabled
centosplus-source/7                       CentOS-7 - Plus Sources                                                disabled
cr/7/x86_64                               CentOS-7 - cr                                                          disabled
epel/x86_64                               Extra Packages for Enterprise Linux 7 - x86_64                         enabled: 13,256
epel-debuginfo/x86_64                     Extra Packages for Enterprise Linux 7 - x86_64 - Debug                 disabled
epel-source/x86_64                        Extra Packages for Enterprise Linux 7 - x86_64 - Source                disabled
epel-testing/x86_64                       Extra Packages for Enterprise Linux 7 - Testing - x86_64               disabled
epel-testing-debuginfo/x86_64             Extra Packages for Enterprise Linux 7 - Testing - x86_64 - Debug       disabled
epel-testing-source/x86_64                Extra Packages for Enterprise Linux 7 - Testing - x86_64 - Source      disabled
extras/7/x86_64                           CentOS-7 - Extras                                                      enabled:    392
extras-source/7                           CentOS-7 - Extras Sources                                              disabled
fasttrack/7/x86_64                        CentOS-7 - fasttrack                                                   disabled
teamviewer/x86_64                         TeamViewer - x86_64                                                    enabled:     77
updates/7/x86_64                          CentOS-7 - Updates                                                     enabled:    240
updates-source/7                          CentOS-7 - Updates Sources                                             disabled
repolist: 24,035

# yum --showduplicates list teamviewer-host
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.bfsu.edu.cn
 * epel: mirrors.yun-idc.com
 * extras: mirrors.ustc.edu.cn
 * updates: mirrors.ustc.edu.cn
Installed Packages         
teamviewer-host.x86_64        15.5.3-0              @/teamviewer-host.x86_64
Available Packages                                
......
teamviewer-host.x86_64        14.2.8352-0           teamviewer
teamviewer-host.x86_64        14.3.4730-0           teamviewer
......
teamviewer-host.x86_64        15.4.4445-0           teamviewer
teamviewer-host.x86_64        15.5.3-0              teamviewer

# yum install teamviewer-host-14.3.4730-0.x86_64

# teamviewer --info

{{< /highlight >}}

封面图片来自 [360º 'VR' Dashboard](https://dribbble.com/shots/3518241-360-VR-Dashboard) <a href="https://dribbble.com/ricardonask"><i class="fa fa-dribbble" aria-hidden="true"></i> Ricardo Nask</a>