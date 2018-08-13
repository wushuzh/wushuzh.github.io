+++
date = "2017-06-04T20:40:13+08:00"
title = "syslog 简介"
showonlyimage = false
image = "/img/blog/syslog-std-in-general/ps_largeconsole.png"
draft = false
weight = 31
tags = ["linux"]
+++

syslog 标准和常见工业实现
<!--more-->

## syslog 是什么
Syslog是被广泛应用在各类设备中( 如打印机、路由器等 )的消息日志标准，它解耦了

- 生成消息日志的软件
- 负责持久存储的系统
- 与日志处理和报表相关的软件

这样从各个异构系统中产生的日志数据就可以通过网络传输汇聚到一个中央仓库。

Syslog作为软件诞生于近40年前，作为Sendmail项目的一部分，并逐渐成为了事实标准，但一直没有任何权威机构当做出正式标准，这也导致了业界产生了很多的各类改进实现，不少还互不兼容。后来IETF先在 RFC 3164 中对现状做了文档描述，并进一步在 RFC 5424 形成正式的标准。

## syslog 含什么
1. 施设代号、严重等级
2. 消息头域，比如进程号/名、时间戳、设备IP地址
3. 日志本身

设施代号，从0到23均用于产生日志的程序的类别划分，不同类别可能对应不同的处理方式，挑几个比较重要的

设施代号 | 关键字   | 描述
----     | ----     | ------
0        | kern     | 内核消息
1        | user     | 用户级别消息
3        | daemon   | 系统守护相关
4 , 10   | authpriv | 都是安全鉴权消息
15       | cron     | 计划任务相关
16 ~ 23  | local?   | 为预留给本地使用 (? 从 0 到 7)

---

等级代号 | 关键字   | 描述
-------- | -------- | -----
0        | emerg    | “内核恐慌”
1        | alert    | 需要立即修复,比如网络断掉
2        | crit     | “崩溃”、“内核转储”、“吐核”，比如备用连接断掉
3        | err      | 错误，非紧急错误，但需要开发和管理员处理
4        | warning  | 警告，比如磁盘使用超过 85%
5        | notice   | 留意，并非错误但也许潜在问题
6        | info     | 普通消息，报告类
7        | debug    | 调试消息

<br />

> 一个便于记忆的英文短句可以辅助记忆消息等级顺序(从轻到重)
>
> "Do I Notice When Evenings Come Around Early"
> 干，我发现暗夜来得早了

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

伴随systemd系统而生，是事件记录的守护进程，总以二进制形式追加记录。利用 journalctl 可以:

- 查看系统引导过程中的日志
- 强大的按需过滤相关日志集中查看
- 将日志内容按照自定义格式(syslog) 或对象(JSON)输出

但二进制文件潜在易损性引起了业界的一些争论，当然系统管理员可以在 Syslog-ng、Rsyslog 和 Systeml-journald 之间做出自由选择。

常用的 journalctl 过滤操作包括:

- 查看当前启动产生的日志 `-b`
- 列出所有记录在案的重启 `--list-boots`
- 查看过去某次重启的日志 `-b -rID`
- 查看某个时间窗口的日志 `--since/--until`
- 查看人类易于理解的时间指代词 "yesterday/today/tomorrow" "1 hour ago"
- 过滤出某个服务的日志 `-u asvc --since today`
- 列出 PID、UID、GID 所有出现过的值 `-F _XID`
- 仅滤出某个 ID 的日志
- 显示内核相关日志记录(dmesg) `-k` 可结合 -b 显示之前某次启动的内核信息
- 显示某个级别的日志记录 -p

用于控制显示的操作包括:

- 省略显示过长的行 `--no-full`
- 将显示直接输出而不是其他 pager 查看器 `--no-pager`
- 将每条日志记录输出为其他各式 例如 `-o json`

显示最后几条日志的操作:

- 显示最后 n 条记录 `-n 20`
- 追踪新生成的日志 `-f`

日常维护的操作:

- 查看磁盘用量 `--disk-usage`
- 删除日志 `--vacuum-size=nG -vacuum-time=nyears`


缩略语解释

> - IETF：互联网工程任务组
> - Rsyslog: the **r**ocket-fast **sys**tem for **log** processing
> - systemd: 用于用户空间初始化和相关管理的系统，和System V或BSD init类似

参考文档

> - https://en.wikipedia.org/wiki/Syslog
> - https://en.wikipedia.org/wiki/Rsyslog
> - https://wiki.archlinux.org/index.php/rsyslog
> - https://en.wikipedia.org/wiki/Syslog-ng
> - https://wiki.archlinux.org/index.php/Systemd#Journal
> - https://en.wikipedia.org/wiki/Systemd#journald
> - Justin Ellingwood (2018-02-20) [How To Use Journalctl to View and Manipulate Systemd Logs](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)

封面图片来自 [Console.Logging Shirt now on Cotton Bureau](https://dribbble.com/shots/1213782-Console-Logging-Shirt-now-on-Cotton-Bureau) <a href="https://dribbble.com/sethakkerman"><i class="fa fa-dribbble" aria-hidden="true"></i> Seth Akkerman</a>
