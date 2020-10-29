+++
date = "2020-01-13T09:56:07+08:00"
title = "ARTS 2020w02"
image = "/img/blog/arts-83-per-week/"
draft = true
weight = 2002
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

[Javarevisited](https://twitter.com/javarevisited) 的新文章 Javin Paul (2020-01-11) [The 2020 Java Developer RoadMap](https://javarevisited.blogspot.com/2019/10/the-java-developer-roadmap.html) 给出了成为 Java 开发者的成长路径图。




## Tip

遇到一个问题：在 archlinux 上通过 vagrant 创建的 kvm 虚拟机不能访问互联网。

问题需要分宿主机和虚拟机两个界面来解决：

1. 对于宿主机 archlinux 上的域名解析排查。

首先明确，浏览器一般使用 DNS-over-HTTPS ，所以系统 DNS 设置和浏览器是否能访问域名无关。

先通过命令行工具测试当前系统的域名解析：

{{< highlight txt >}}
$ drill bing.com
Error: error sending query: No (valid) nameservers defined in the resolver

$ ls -l /etc/resolv.conf 
-rw-r--r-- 1 root root 65 Nov 14 00:23 /etc/resolv.conf

$ cat /etc/resolv.conf 
# Resolver configuration file.
# See resolv.conf(5) for details.

$ getent hosts bing.com
2620:1ec:c11::200 bing.com
{{< /highlight >}}

- `drill` 命令解析域名失败，因为其运行只依赖于配置文件 `/etc/resolv.conf` 中设置的域名服务器，之后它自己独立完成域名查询的所有交互。
- `getent` 命令解析域名成功，因为其配置文件 `/etc/nsswitch.conf` 中设定通过系统服务 `systemd-resolved` 中设定的 DNS 来解析域名。

{{< highlight txt >}}
$ cat /etc/nsswitch.conf  |grep hosts
# hosts: files mymachines myhostname resolve [!UNAVAIL=return] dns
hosts: files libvirt libvirt_guest mymachines myhostname resolve [!UNAVAIL=return] dns

$ systemctl status systemd-resolved
● systemd-resolved.service - Network Name Resolution
   Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-01-17 17:30:33 CST; 2 days ago
     Docs: man:systemd-resolved.service(8)
           https://www.freedesktop.org/wiki/Software/systemd/resolved
           https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
           https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
 Main PID: 531 (systemd-resolve)
   Status: "Processing requests..."
    Tasks: 1 (limit: 9339)
   Memory: 16.1M
   CGroup: /system.slice/systemd-resolved.service
           └─531 /usr/lib/systemd/systemd-resolved
{{< /highlight >}}

`systemd-resolved` 的配置文件为 `/etc/systemd/resolved.conf` 

{{< highlight txt >}}
cat resolved.conf 
#  This file is part of systemd.
# ......
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
# See resolved.conf(5) for details

[Resolve]
#DNS=
#FallbackDNS=1.1.1.1 9.9.9.10 8.8.8.8 2606:4700:4700::1111 2620:fe::10 2001:4860:4860::8888
#Domains=
#LLMNR=yes
#MulticastDNS=yes
#DNSSEC=allow-downgrade
#DNSOverTLS=no
#Cache=yes
#DNSStubListener=yes
#ReadEtcHosts=yes
{{< /highlight >}}

若希望查询 resolved 的设定情况，通过 `resolvectl` 查询：

{{< highlight txt >}}
$ resolvectl status
Global
       LLMNR setting: yes
MulticastDNS setting: yes
  DNSOverTLS setting: no
      DNSSEC setting: allow-downgrade
    DNSSEC supported: no
Fallback DNS Servers: 1.1.1.1
                      9.9.9.10
                      8.8.8.8
                      2606:4700:4700::1111
                      2620:fe::10
                      2001:4860:4860::8888
          DNSSEC NTA: 10.in-addr.arpa
                      16.172.in-addr.arpa
......
                      corp
                      d.f.ip6.arpa
                      home
                      internal
                      intranet
                      lan
                      local
                      private
                      test

Link #id (interface-name) for example,

Link 3 (wlp0s20f0u1)
      Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
DefaultRoute setting: yes
       LLMNR setting: yes
MulticastDNS setting: no
  DNSOverTLS setting: no
      DNSSEC setting: allow-downgrade
    DNSSEC supported: no
  Current DNS Server: 192.168.43.1
         DNS Servers: 192.168.43.1

{{< /highlight >}}

对于 `systemd-resolved` DNS 的配置有几种模式，这里采用 systemd DNS stub 文件的形式，即创建链接文件 `/etc/resolv.conf` ，指向 `/run/systemd/resolve/stub-resolv.conf`，其中配置 DNS 为 127.0.0.53

{{< highlight txt >}}
cat /run/systemd/resolve/stub-resolv.conf

nameserver 127.0.0.53
options edns0

sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

{{< /highlight >}}

之后 drill 指令可以查询成功。

{{< highlight txt >}}
$ drill bing.com
;; ->>HEADER<<- opcode: QUERY, rcode: NOERROR, id: 15778
;; flags: qr rd ra ; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0 
;; QUESTION SECTION:
;; bing.com.	IN	A

;; ANSWER SECTION:
bing.com.	3080	IN	A	204.79.197.200
bing.com.	3080	IN	A	13.107.21.200

;; AUTHORITY SECTION:

;; ADDITIONAL SECTION:

;; Query time: 52 msec
;; SERVER: 127.0.0.53
;; WHEN: Mon Jan 20 14:39:07 2020
;; MSG SIZE  rcvd: 58
{{< /highlight >}}

对于基于 kvm 的虚拟机，vagrant 默认创建一个名为 vagrant-libvirt 的 192.168.121.0/24 网络，并设置了 NAT。 


{{< highlight txt >}}

$ virsh net-dumpxml vagrant-libvirt
<network connections='1' ipv6='yes'>
  <name>vagrant-libvirt</name>
  <uuid>25e0aa29-6f96-471c-bf27-9bf270a47705</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr2' stp='on' delay='0'/>
  <mac address='52:54:00:9c:f6:f9'/>
  <ip address='192.168.121.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.121.1' end='192.168.121.254'/>
    </dhcp>
  </ip>
</network>
{{< /highlight >}}

## Share

吴军《智能时代》第一章，数据基石

{{< figure src="/img/blog/arts-83-per-week/data-foundation.jpg" title="智能时代-数据基石" >}}

封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>