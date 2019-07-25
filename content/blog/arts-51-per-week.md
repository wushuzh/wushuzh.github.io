+++
date = "2019-06-10T13:50:54+08:00"
title = "ARTS 2019w24"
showonlyimage = false
image = "/img/blog/arts-51-per-week/user-data.webp"
topImage = "/img/blog/arts-51-per-week/user-data.gif"
draft = false
weight = 1924
tags = ["ARTS", "Union-Find"]
+++

提高学习效率，少做无用功的一些技巧
<!--more-->

## Algorithm

[leetcode #721 Accounts Merge](https://leetcode.com/problems/accounts-merge/) 以 email 作为依据合并冗余账户——此题的关键要梳理好三个概念：

1. 用户名：用户自行定义，现实世界中不同的人都会有重名现象，更何况同一用户会有小号——当下多数应用都不允许重名，但这样的要求多是从使用者角度的考量——要辅助用户互动中易于相互识别。但本题对“小号现象”理想化了一下：要求只要是同一用户，用户名一定一样；
2. 邮箱地址：同一用户可以有多个邮箱使用在不同场景（工作邮箱，个人邮箱，国内备用邮箱……）；
3. 账户: 如果同一用户通过不同邮箱注册，应用系统默它们是不同的账户，但如果账户存在辅助信息（比如备用邮箱），我们就可以将上面的同名账户关联起来——方便更合理的资源分配、信息整合等，最后返回的结果，用户名依然可能重复，但所有邮箱都只能出现一次——属于某一个用户；

Union-Find 的实现要点：

- 引入一个 dict，利用 email 的唯一性作为 key，value 为用户名，形成最终结果时查找会比较方便；
- email 是 Union-Find 数据结构的核心，先将已知在同一账户下 email 做 union 操作，然后在整合其他用户邮箱时，因为之前的邮箱关联，会形成最终的“独立”用户邮箱组；
- 找出这些“独立”组利用 find_root 操作：只要 root 相同就在同一组，这个 root 因为有唯一性，可作为 dict 的 key；
- python 的 collections 包中的 defaultdict 类特别适合作为获取独立用户的中转数据类型，最后配合 python 的 list comprehensions 和最开始引入的 email2name 字典返回题目要求结果；

{{< highlight python "linenos=inline, hl_lines=10" >}}
def accounts_merge_using_union_find(accounts: List[List[str]]) -> List[List[str]]:

    uf = UnionFind()
    email_to_name = dict()

    for account in accounts:
        name = account[0]
        for email in account[1:]:
            email_to_name[email] = name
            uf.union(email, account[1])

    return [[email_to_name[pmail]] + sorted(emails) for (pmail, emails) in uf.groups().items()]

import collections


class UnionFind:
    def __init__(self):
        self.parent = dict()

    # omit method find_root and union

    def groups(self):
        merge_accounts = collections.defaultdict(list)
        for email in self.parent.keys():
            merge_accounts[self.find_root(email)].append(email)
        return merge_accounts

{{< /highlight >}}

## Review

如何搭建一个简单的 kubernetes 实验集群 ？

- 纯手动利用 VirtualBox 在笔记本搭建；
- 半手工利用 ansible 优化 k8s 安装过程；
- 全自动利用 terraform 优化 OS 并集成整体安装过程；

<img alt="Kubernetes trial" src="/img/blog/arts-51-per-week/k8s-install.jpg" class="img-responsive">

## Tip

从 Medium 上看到 Marko Lukša 的一个 bash 技巧介绍：[Bash trick: Repeat last command until success](https://medium.com/@marko.luksa/bash-trick-repeat-last-command-until-success-750a61c43c8a)

有时 pod 启动有一个过程，此时执行查看各种查看，连接等操作都会失败。可以将下面的函数放入 .bashrc 中，自动帮我们重复上一条命令，直到被访问对象或服务变为可用状态，将我们计划执行的命令执行成功。原理就是利用 [fc (process the command history list)](https://www.systutorials.com/docs/linux/man/1p-fc/) 获取上次运行的指令，然后放入循环：只要命令执行不成功，返回值不为 0，休眠一下，稍后再试。

应用场景可能有：

查看：

- `kubectl get cm foo -o yaml`
- `tailf /var/log/some.log`

等待服务：

- `curl http://barsvc.default.svc.cluster.local:svcport`
- `kubectl exec -it mypod bash`
- `ssh me@host-restarting`

<br/>

**特别不理解的是函数第二行的 tail 为什么要列出最后两条历史指令，然后再转给 head 取第一条指令。**

{{< highlight bash "linenos=inline">}}
rpt () {
  CMD=$(fc -ln | tail -2 | head -1)
  echo "repeating until success: $CMD"
  until $CMD
  do
    sleep 1
  done
}
{{< /highlight >}}

## Share

看了一个youtube的推荐视频 [how to study smart by Marty Lobdell](https://youtu.be/IlU-zDU6aQ0) Marty Lobdell 教授为新生普及如何高效学习：误区和技巧。其主业是心理学，课讲的风趣，正好我自己最近也在集中学习，要有意识地将下面记录的要点进行实践。

- 心态上，要关注效率，保持积极心态，发现自己走神后，主动休息——像锻炼肌肉一样，逐渐增强学习长度，尝试每天完结后给予自己奖励；  
- 行动上，为自己营造一个学习角：专属台灯，正姿而专注，走神就关灯起身，主动休息；  
- 方法上，重概念，轻事实。将概念连接到已知的知识树上，能用自己的语言解释才算入门，对事实性知识，用各种各样的记忆术: 缩略、谐音、谚语、做诡异的可视花；
- 读书时，花5分钟审视全章(survey)内容，提出一些问题(question)，然后阅读(read)，背诵(recite)，回顾(review): SQ3R；
- 下课后，立刻花 5 分钟复习内容，参与小组讨论，不具备条件的对着空椅子讲述，好好休息

封面图片来自 [User Data Animated Illustration](https://dribbble.com/shots/3211423-User-Data-Animated-Illustration) <a href="https://dribbble.com/Ramotion"><i class="fa fa-dribbble" aria-hidden="true"></i> Ramotion</a>