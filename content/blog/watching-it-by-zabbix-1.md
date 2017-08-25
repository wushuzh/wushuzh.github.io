+++
date = "2017-08-18T20:51:17+08:00"
title = "Zabbix 综合监控"
showonlyimage = false
image = "/img/blog/watching-it-by-zabbix-1/power_system_icon_1x.jpg"
draft = false
weight = 71
+++

其名和监控，大概能有个十万八千里的距离？
<!--more-->

## 简介

最近学习了一些综合监控的开源解决方案，并尝试了其中一个功能比较均衡的产品 Zabbix。除了[维基百科](https://en.wikipedia.org/wiki/Zabbix)，还可以看下其创始人 Alexei Vladishev 的介绍

- [Zabbix: Free Software that helps](https://www.slideshare.net/xsbr/zabbix-by-alexei-vladishev-fisl12-2011)
- [Open Source Enterprise Monitoring with Zabbix](https://web.archive.org/web/20120226220044/http://www.netways.de/uploads/media/Alexei_Vladishev_Open_Source_Monitoring_with_Zabbix.pdf)

关于其名字的由来，推测应该是这样的：几位工程师终于完成了当天的编码任务，现在产品的核心功能已经很像那么回事了。大家都觉得应该利用茶歇的时间，考虑一下今后推广这个牛X产品时该如何称呼它，于是就开始了头脑风暴式的讨论。

但没想到进入了这样一个尴尬的循环：想出一个名，一查已被注册，又想一个，一查又已被使用。好像是所有符合监控本身含义的酷名字都被用掉了。尝试了许久过后，终于大家的耐心被彻底击溃，然后开始像跟这个世界有过节似得胡乱编名字，最终找到了一个完全无意义的单词 Zabbix，而网上一搜——果然完全没有人用——连搜索引擎返回的记录数都为 0，完美，就这个了，碎觉……

## 总体

记录一下我的搭建环境和基本过程：

1. 创建一台虚拟机 S，安装配置 LAMP、Zabbix 核心服务器及其前端
2. 创建一台虚拟机 V，安装 LAMP 和 Zabbix 代理，目标是收集其服务器上的相关指标
3. 创建一台虚拟机 D，安装 Docker 和 Zabbix 代理，和 V 相比，这里还要外加收集此服务器上运行容器的耗费的系统资源指标
4. 定义一些自有指标，调试并通过网页查看实时结果

> 搭建的具体步骤，参见 Vadym Kalsin (2016-11-09) [How To Install and Configure Zabbix to Securely Monitor Remote Servers on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-zabbix-to-securely-monitor-remote-servers-on-centos-7)
>
> 因为 Vadym Kalsin 将安装配置写得非常细致和完善，后面只记录我在自己的环境下特定的操作配置和踩到的坑。

### 追加操作

为了快速搭建，直接使用 CentOS 7 的 Generic Cloud 的 qcow2 的 image。借助 [giovtorres/kvm-install-vm](https://github.com/giovtorres/kvm-install-vm) 脚本，配好 Artifactory CentOS 的 Repository。基本 3 - 5 分钟内就能建好三台待用的虚拟机。

在虚拟机 S 和 V 上安装 LAMP，比较简答——按照 Mitchell Anicas (2014-07-21) [How To Install Linux, Apache, MySQL, PHP (LAMP) stack On CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7) 安装配置 Apache, MySQL (MariaDB) 和 PHP 即可。

> 记好 MariaDB 的 root 密码，后面为 Zabbix 初始化数据库时要用到

### 踩到的坑

配置 Zabbix 服务器的过程主要涉及两方面：

- 数据库的创建，专属用户，初始化表结构导入，同时更新 zabbix_server.conf 告知 DB 密码
- PHP 服务端则需要配置时区

但之后你试图启动 zabbix-server 的时候却有可能碰到下面的错误。

{{< highlight console >}}
$ systemctl status zabbix-server
  Job for zabbix-server.service failed
    because a configured resource limit was exceeded.
  See "systemctl status zabbix-server.service"
    and "journalctl -xe" for details.

# /var/log/zabbix/zabbix_server.log
Starting Zabbix Server. Zabbix 3.0.10 (revision 70208).
****** Enabled features ******
SNMP monitoring:           YES
IPMI monitoring:           YES
Web monitoring:            YES
VMware monitoring:         YES
SMTP authentication:       YES
Jabber notifications:      YES
Ez Texting notifications:  YES
ODBC:                      YES
SSH2 support:              YES
IPv6 support:              YES
TLS support:               YES
******************************
using configuration file: /etc/zabbix/zabbix_server.conf
cannot set resource limit: [13] Permission denied
cannot disable core dump, exiting...
{{< /highlight >}}

此问题是由于系统 coredump 设置，SELinux 相关策略以及 Zabbix 启动检测的冲突引起。欲知详情，配合 Zabbix 的 JIRA [zabbix-server can not start on rhel 7.1](https://support.zabbix.com/browse/ZBX-10542) 和 [An Introduction to SELinux on CentOS 7](https://www.digitalocean.com/community/tutorial_series/an-introduction-to-selinux-on-centos-7) 可做深入解读。

粗暴的解决方法可以禁掉 SELinux ，但这么做毕竟不专业，所以至少先安装了 setroubleshoot 并启动相应服务：

{{< highlight console >}}

sudo yum install setroubleshoot
sudo systemctl restart auditd.service
  # =====================================================
  # if you found error when start auditd
  #   Failed to restart auditd.service: Operation refused,
  #       unit auditd.service may be requested by dependency only.
  #   See system logs and
  #       'systemctl status auditd.service' for details.
  # then execute following instead
  #   'sudo service auditd restart'
  # check more details on
  #    https://access.redhat.com/solutions/2664811
  # =====================================================

{{< /highlight >}}

然后根据```/var/log/messsage```中的提示为本地创建例外规则，放行我们可预期的服务。
操作参见:

- [SELinux : Use SETroubleShoot](https://www.server-world.info/en/note?os=CentOS_7&p=selinux&f=8)
- [鸟哥的 Linux 私房菜 第七章、网络安全与主机基本防护：限制端口, 网络升级与 SELinux](http://cn.linux.vbird.org/linux_server/0210network-secure_4.php)
- [Red Hat bugzilla 1323818 Zabbix agent fails to start due to being unable to disable coredumps](https://bugzilla.redhat.com/show_bug.cgi?id=1323518)
- [Red Hat bugzilla 1393332 unable to start zabbix agent after upgrade to RHEL 7.3](https://bugzilla.redhat.com/show_bug.cgi?id=1393332)

### 访问VM网络

成功启动了 Zabbix 和 httpd 服务，你需要打开浏览器，指向 Zabbix 服务所在虚拟机的地址```http://your_zabbix_server_ip_addr/zabbix/```，做 Zabbix 的首次启动配置(这次用到了 MariaDB 的 Zabbix 用户的密码)。

而因为我使用一台性能强劲一些的远程服务器(而非自己的笔记本)做虚拟机的宿主服务器，而后续可能有一些需要访问启动在虚拟机上的网页来实现 Zabbix 配置，所以提前在宿主服务器上配置好 vnc 服务及防火墙。

{{< highlight bash >}}
$ sudo yum install tigervnc-server
$ sudo firewall-cmd --get-services
$ sudo firewall-cmd --permanent --zone=public --add-service=vnc-server
{{< /highlight >}}
<br />

### 安全秘钥通讯

之后就是配置被监控节点，以虚拟机 V 为例，为了使得监控通信在安全的信道中完成，请遵照教程中指令为这台虚拟机生成配置一个 PSK 秘钥。

如果你在虚拟机 V 或 D 上面，启动 zabbix-agent 服务时遇到了之前在 虚拟机 S 上类似的错误，用和之前相同的方法解决。反正我试用了 centos 7 2017-08 的版本的 image ，这个和 SELinux 有关的问题仍然能复现。

{{< highlight console >}}

# /var/log/zabbix/zabbix_agentd.log
Starting Zabbix Agent [Zabbix server]. Zabbix 3.0.10 (revision 70208).
**** Enabled features ****
IPv6 support:          YES
TLS support:           YES
**************************
using configuration file: /etc/zabbix/zabbix_agentd.conf
cannot set resource limit: [13] Permission denied
cannot disable core dump, exiting...

{{< /highlight >}}

成功启动 zabbix-agent 后，就可以回到虚拟机 S 上利用网页添加这个被监控的主机了。记得使用 PSK 通信兵配置之前准备好的 PSK，最后添加监控模板，就可以在网页上查看产生的性能和告警。

关于监控自定义的性能指标，再开一篇另作说明。

参考文档

> - Sadequl Hussain (2014-11-25) [How To Install and Configure VNC Remote Access for the GNOME Desktop on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-remote-access-for-the-gnome-desktop-on-centos-7)
> - Justin Ellingwood (2015-06-18) [How To Set Up a Firewall Using FirewallD on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7)

封面图片来自 [Energy System Icon 02](https://dribbble.com/shots/1044323-Energy-System-Icon-02) <a href="https://dribbble.com/Kingyo"><i class="fa fa-dribbble" aria-hidden="true"></i> Kingyo</a>
