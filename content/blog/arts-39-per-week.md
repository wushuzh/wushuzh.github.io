+++
date = "2019-03-18T19:32:29+08:00"
title = "ARTS 2019w12"
showonlyimage = false
image = "/img/blog/arts-39-per-week/overload.webp"
topImage = "/img/blog/arts-39-per-week/overload.gif"
draft = false
weight = 1913
tags = ["ARTS"]
+++

使用并查集，找寻无向图中冗余的连接
<!--more-->

## Algorithm

[leetcode #684 Redundant Connection](https://leetcode.com/problems/redundant-connection/)：给定一组连接关系（连接无方向），去除其中的冗余连接，使得最终图形不会形成[环(回路)](https://en.wikipedia.org/wiki/Cycle_(graph_theory))

本题用 Union-Find 数据结构解决，主要思路是按给定顺序在 UF 数据结构中做链接，当此次连接操作无效——没有连接未知节点，即找到了冗余连接。

此题首先注意：

- 节点 id 题目中已经给定，是从 1 到 n —— 这样数组的索引就不能直接作为节点 id，考虑用更加直接的 dict 数据结构；
- union 操作的返回值设定为布尔型，表示此次链接是否真正做了链接；

{{< highlight python "linenos=inline, hl_lines=4" >}}
def find_redundant_conn_by_union_find(edges: List[List[int]]) -> List[int]:
    uf = UnionFind(len(edges))
    for a, b in edges:
        if not uf.union(a, b):
            return [a, b]
{{< /highlight >}}

## Review

[Structured Logging: The Best Friend You’ll Want When Things Go Wrong](https://engineering.grab.com/structured-logging) ：讲述了 Grab 从开始直接使用第三方的 SaaS 日志存储服务，到后来开发自己的日志库，使用 Elastic Stack 作为存储后端，实现了更丰富的查询，更灵活的配置，更精准的复原追踪的历程。

{{< figure src="/img/blog/arts-39-per-week/grab-log.jpg" title="Grab log" >}}

## Tip

两个有业务关联的组：财务(account)和销售(sales)

1. 各自组内的人员应该都有查阅，修改，删除的权限，特别是组内人员创建的文件显示组别应该是组名: GUID实现；
2. 跨组人员应该有查阅权限: ACL实现；

{{< highlight bash >}}
# 0. setup env
# add 4 users with supplementary 2 groups 
groupadd account
useradd -G account adams
useradd -G account abcd

groupadd sales
useradd -G sales sams
useradd -G sales stevens

# setup share folder
mkdir -p /data/{account,sales}
chmod 770 /data/*
chgrp account /data/account
chgrp sales /data/sales

ls -l /data/*
|| drwxrwx---  2 root sales   4096 Jul 18 09:18 sales
|| drwxrwx---  2 root account 4096 Jul 18 09:18 account

# 1. group member cannot modify each other files
su - stevens -c "touch /data/sales/stevens_file"
su - stevens -c "ls -l /data/sales/stevens_file"
|| -rw-rw-r-- 1 stevens stevens 8 Jul 18 09:19 /data/sales/stevens_file
su - sams -c "echo sams > /data/sales/stevens_file"
|| -bash: /data/sales/stevens_file: Permission denied

# fix by setGID: change default group of new file as same as folder group
chmod g+s /data/*
su - sams -c "echo sams > /data/sales/sams_file"
su - sams -c "ls -l /data/sales/sams_file"
|| -rw-rw-r-- 1 sams sales 5 Jul 18 09:51 /data/sales/sams_file
su - stevens -c "echo stevens >> /data/sales/sams_file"
# write successfully

# 2. But account group members cannot read sales team files
su - abcd -c "cat /data/sales/stevens_file"
|| cat: /data/sales/stevens_file: Permission denied

# fixed by ACL add account group for sales folder
setfacl -m g:account:rx /data/sales
getfacl /data/sales
|| # file: sales
|| # owner: root
|| # group: sales
|| # flags: -s-
|| user::rwx
|| group::rwx
|| group:account:r-x
|| mask::rwx
|| other::---
su - abcd -c "cat /data/sales/stevens_file"
|| sams
|| stevens

{{< /highlight >}}

## Share

[Intro to SSH and SSH Keys](https://youtu.be/mF6J-VQHPxA) 是一个介绍 SSH KEY 的视频。

{{< figure src="/img/blog/arts-39-per-week/ssh-host-and-user-key.jpg" title="SSH KEYS" >}}

SSH Key 用于标识服务器

- 任何一台服务器上的 SSHD 服务都要生成自己专属的 Server key (一对，公钥+私钥)用于辨识自己 `/etc/ssh/ssh_host_<rsa>_key[.pub]`和传输数据的加密；
- 客户端首次连接服务器时，服务器会被要求传输公钥到客户端，客户端 ssh 连接工具从中提取出相应的 fingerprint 要求客户端确认这是自己希望连接的服务器；
- 正规流程时我们联系管理员，提供 fingerprint 要求确认这台服务器可信，将其保存到客户端 `/etc/ssh/known_hosts` 及 `$HOME/.ssh/known_hosts` 里

SSH Key 用于标识用户

- 用户在本地生成一对自己的专属 User Key （一对，公钥+私钥）`$HOME/.ssh/id_ras[.pub]`;
- 用户如果能够在对端服务器上添加自己的公钥到 `~/.ssh/authorized_keys` ，比如 `ssh-copy-id`，就可以实现自动登陆；

日常使用不规范的操作：

1. 我们首次连接服务器时，很少去找管理员核实；
2. 没有专门用户保存 ssh key inventory；
3. ssh key 不设轮换机制；
4. 很久以前通过不强壮算法生成的 key 大量存在；
5. 缺少管理，即使离职员工仍可以通过 key 访问系统，然后通过中转登陆更多系统；


封面图片来自 [Guidline Overload](https://dribbble.com/shots/3418553-Guidline-Overload) <a href="https://dribbble.com/Tmotion"><i class="fa fa-dribbble" aria-hidden="true"></i> Tigran Manukyan</a>