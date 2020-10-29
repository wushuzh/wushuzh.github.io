+++
date = "2020-04-20T22:47:33+08:00"
title = "ARTS 2020w16"
image = "/img/blog/arts-97-per-week/"
draft = true
weight = 2016
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

[#876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) 给定一个非空的单链表，要求返回处于原链表中间元素的那个节点，如果有两个节点处于中间，返回处于后面的那个节点即可。

对整个链表的一次全遍历是无可避免的，可以使用快/慢双指针，快指针一次走两个，慢指针一次走一步。

{{< highlight python >}}
def get_mid_v1(head: ListNode) -> ListNode:
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
{{< /highlight >}}

## Review

## Tip

## Share

3个月没有到办公室，留在那里的 PC 落了灰。长时间未使用，对 archlinux 很追新的系统“比较”致命。

archlinux 要求安装任何新软件，要连带做一下系统的升级——否则导致依赖性问题，这样就会导致更新一堆巨量的软件包，升级时极大可能出现向前兼容的问题。平时处理单个问题，

遇到的第一个问题：密钥的更新
遇到的第二个问题：linux版本太新，无线网卡追不上其更新速度

(507/507) checking package integrity                                                                               [####################################################################] 100%
error: containerd: signature from "Santiago Torres-Arias <santiago@archlinux.org>" is unknown trust
:: File /var/cache/pacman/pkg/containerd-1.3.3-1-x86_64.pkg.tar.zst is corrupted (invalid or corrupted package (PGP signature)).
Do you want to delete it? [Y/n] n
error: failed to commit transaction (invalid or corrupted package)
Errors occurred, no packages were upgraded.



https://bbs.archlinux.org/viewtopic.php?id=252920


https://wiki.archlinux.org/index.php/Pacman/Package_signing

https://www.gnupg.org/gph/en/manual.html#AEN385


linux版本回退

0. 确认网卡驱动当前支持的 kernel 版本  https://aur.archlinux.org/packages/rtl8812au-dkms-git/

描述中期望版本 rtl8812AU chipset driver with firmware v5.6.4.2
实际版本:
pacman -Qn linux
linux 5.6.6.arch1-1


1. 通过 https://wiki.archlinux.org/index.php/Android_tethering 联网
2. 死马当活马医治，强行安装最新版本的 aur rtl8812au-dkms-git

重启后生效，

不然就的考虑回退 archlinux ，
这属于非常规操作，安装 AUR downgrader 工具 https://wiki.archlinux.org/index.php/downgrading_packages
pacman -Qm

sudo downgrader linux

封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>