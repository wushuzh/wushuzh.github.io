+++
date = "2017-06-04T20:40:13+08:00"
title = "syslog 标准及其实现"
showonlyimage = false
image = "/img/blog/dual-os-5-wifi-ap/freewifi.png"
draft = true
weight = 66
+++

syslog 标准及其实现
<!--more-->

# syslog 标准及其实现

## syslog是什么
Syslog是被广泛应用在各类设备中( 如打印机、路由器等 )的消息日志标准，它解耦了
- 生成消息日志的软件
- 负责持久存储的系统
- 与日志处理和报表相关的软件

这样从各个异构系统中产生的日志数据就可以通过网络传输汇聚到一个中央仓库。Syslog作为软件诞生于近40年前，作为Sendmail项目的一部分，并逐渐成为了事实标准，但一直没有任何权威机构当做正式标准发布，这也导致了太多的各类改进实现，不少还互不兼容。后来IETF先在 RFC 3164 中对现状做了文档描述，并进一步在 RFC 5424 形成正式的标准


## syslog含什么
1. 施设代号、严重等级
2. 消息头域，比如进程号/名、时间戳、设备IP地址
3. 日志本身

设施代号，从0到23均用于产生日志的程序的类别划分，不同类别可能对应不同的处理方式，挑几个比较重要的

设施代号 | 描述
----|------
0 | 代表内核
1 | 代表用户
3 | 系统守护相关
4 , 10 | 都是安全鉴权
9 | 时钟和时间同步
15 | 计划任务相关
16 ~ 23 | 为预留给本地使用 local 0 ~ 7

---

等级代号 | 关键字 | 描述
--------|--------|-----
0 | emerg | “内核恐慌”
1 | alert | 需要立即修复
2 | crit | “崩溃”、“内核转储”、“吐核”
3 | err | 错误
4 | warning | 警告
5 | notice | 留意
6 | info | 普通信息
7 | debug | 调试

## Syslog-ng 和 Rsyslog
Syslog-ng始于1998年，作为对syslog协议的开源实现，增加大量丰富的功能。在提供开源版本的同时，也提供了商业版本。
Rsyslog则要晚一点，开始于2004年，对于其名字的官方解释是“像火箭一样的日志处理”，同样实现了syslog协议，并添加了如过滤，配置，传输等增强功能。按照作者 Rainer Gerhards的说法，“（Rsyslog）作为一个重量级玩家可以防止垄断并提供了选择的自由”。

他们的主要功能有：

- 毫秒级时间粒度以及时区
- 增加对中继服务器名的记录，以便追踪某一消息的上报路径
- 使用可靠的网络传输TCP
- 直接将记录存储至数据库
- 做多个日志源的关联，整合成一个负责事件(Syslog-ng OSE3.2)
- 当日志接受端未就位时，做本地缓存的操作
- 对systemd journal输入输出的完整支持
- RELP、GSS-API/TLS

## Systemd-journald
伴随systemd系统而生，是事件记录的守护进程，总以追加的形式将数据做二进制存储。但二进制文件潜在易损性引起了业界的一些争论，当然系统管理员可以在Syslog-ng、Rsyslog和Systeml-journald之间做出自由选择。

缩略语解释
> - IETF：互联网工程任务组
> - Rsyslog: the **r**ocket-fast **sys**tem for **log** processing
> - systemd: 用于用户空间初始化和相关管理的系统，和System V或BSD init类似

参考文档
> - https://en.wikipedia.org/wiki/Syslog
> - https://en.wikipedia.org/wiki/Rsyslog
> - https://en.wikipedia.org/wiki/Syslog-ng
> - https://wiki.archlinux.org/index.php/Systemd#Journal
> - https://en.wikipedia.org/wiki/Systemd#journald

文中图片来源网络
