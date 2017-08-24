+++
date = "2017-08-18T20:51:17+08:00"
title = "Zabbix 综合监控"
showonlyimage = false
image = "/img/blog/watching-it-by-zabbix-1/power_system_icon_1x.jpg"
draft = false
weight = 65
+++

其名和监控，大概能有个十万八千里的距离？
<!--more-->

## 简介

最近了解了一下系统综合监控的开源解决方案，尝试了其中一个看上去功能很均衡的产品 Zabbix。除了[维基百科](https://en.wikipedia.org/wiki/Zabbix)，还可以看下其创始人 Alexei Vladishev 的介绍

- [Zabbix: Free Software that helps](https://www.slideshare.net/xsbr/zabbix-by-alexei-vladishev-fisl12-2011)
- [Open Source Enterprise Monitoring with Zabbix](https://web.archive.org/web/20120226220044/http://www.netways.de/uploads/media/Alexei_Vladishev_Open_Source_Monitoring_with_Zabbix.pdf)

关于其名字的由来，感觉是这样一个典型场景：想出一个名，一查已被注册，又想一个，一查又已被使用。然后胡乱编了现在这个名，网上一搜——果然完全没有人用——搜索记录为 0 ，于是就将其起名为 Zabbix

## 总体

记录一下我的搭建环境和基本过程：

1. 创建一台虚拟机 S，安装并配置 Zabbix 核心服务器、数据库及网页前端
2. 创建一台虚拟机 V，安装 LAMP 和 Zabbix 代理，收集其相关指标
3. 创建一台虚拟机 D，安装 Docker 和 Zabbix 代理，收集 Docker 的相关指标
4. 定义自有指标，调试并通过网页查看实时结果

为了快速搭建，直接使用 CentOS 7 的 Generic Cloud 的 qcow2 的 image。借助 [giovtorres/kvm-install-vm](https://github.com/giovtorres/kvm-install-vm) ，公司内部搭建 Artifactory ，包含 CentOS Repository，将配置文件拷贝到各台机器上。基本 3 分钟之内就能创建好三台待用的虚拟机。

首先在虚拟机 S 和 V 上安装 LAMP，按照 Mitchell Anicas (2014-07-21) [How To Install Linux, Apache, MySQL, PHP (LAMP) stack On CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7) 安装配置 Apache, MySQL (MariaDB) 和 PHP。

> 记好 MariaDB 的 root 密码

因为我使用了一台远程服务器做虚拟主机，而后续又有不少通过网页的配置，所以要在这些服务器上配置 vnc 服务。详情参见 Sadequl Hussain (2014-11-25) [How To Install and Configure VNC Remote Access for the GNOME Desktop on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-remote-access-for-the-gnome-desktop-on-centos-7) 以及防火墙设置


stop firewall enable vnc access
  https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7
  sudo firewall-cmd --permanent --zone=public --add-service=vnc-server



install zabbix according to do

but zabbix cannot start due to selinux
    https://support.zabbix.com/browse/ZBX-10542

    sudo yum install setroubleshoot

    https://www.digitalocean.com/community/tutorials/an-introduction-to-selinux-on-centos-7-part-1-basic-concepts
    https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Which_Log_File_is_Used.html

    https://www.server-world.info/en/note?os=CentOS_7&p=selinux&f=8

        [centos@zabbix-target log]$ sudo systemctl restart auditd.service
        Failed to restart auditd.service: Operation refused, unit auditd.service may be requested by dependency only.
        See system logs and 'systemctl status auditd.service' for details.

        service auditd restart

https://support.zabbix.com/browse/ZBX-10542

https://bugzilla.redhat.com/show_bug.cgi?id=1323518

https://bugzilla.redhat.com/show_bug.cgi?id=1393332

http://cn.linux.vbird.org/linux_server/0210network-secure_4.php

Job for zabbix-server.service failed because a configured resource limit was exceeded. See "systemctl status zabbix-server.service" and "journalctl -xe" for details.


epel python-pip

pip install docker-py

封面图片来自 [Energy System Icon 02](https://dribbble.com/shots/1044323-Energy-System-Icon-02) <a href="https://dribbble.com/Kingyo"><i class="fa fa-dribbble" aria-hidden="true"></i> Kingyo</a>
