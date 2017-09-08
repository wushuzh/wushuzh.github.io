+++
date = "2017-09-05T22:50:33+08:00"
title = "洪水报文，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-flood-logs/taxes.png"
draft = false
weight = 47
+++

如何过滤无用的系统日志……
<!--more-->

["syslog 标准及其实现"]({{< relref "blog/syslog-std-in-general.md" >}}) 中最后提到了一种 syslog 的实现:“快火箭日志”。本篇就针对它说说遇到“洪水报文消息”的处理方法。

> 所谓洪水报文消息，就是系统日志被大量重复信息充斥，导致正常日志被淹没其中，容易被错过忽略，影响系统安全审计。

## 现象描述

RHEL7 中 /var/log/ 目录下 messages 和 secure 两个文件中存在大量的相似的登录会话日志。每秒都出现大量的会话打开、关闭，从 root 转换到 root 等日志。

{{< highlight console >}}
# tailf secure
... HH:MM:31 ... su: pam_unix(su-l:session): session closed for user root
... HH:MM:32 ... su: pam_unix(su-l:session): session opened for user root by (uid=0)
... HH:MM:32 ... su: pam_unix(su-l:session): session closed for user root
... HH:MM:33 ... su: pam_unix(su-l:session): session opened for user root by (uid=0)
... HH:MM:34 ... su: pam_unix(su-l:session): session closed for user root
... HH:MM:36 ... su: pam_unix(su-l:session): session opened for user root by (uid=0)
... HH:MM:37 ... su: pam_unix(su-l:session): session opened for user root by (uid=0)
... HH:MM:37 ... su: pam_unix(su-l:session): session closed for user root

# tailf messages
... HH:MM:32 ... su: (to root) root on none
... HH:MM:32 ... systemd: Started Session c802966 of user root.
... HH:MM:32 ... systemd: Starting Session c802966 of user root.
... HH:MM:33 ... su: (to root) root on none
... HH:MM:33 ... systemd: Started Session c802967 of user root.
... HH:MM:33 ... systemd: Starting Session c802967 of user root.
... HH:MM:34 ... su: (to root) root on none
... HH:MM:34 ... systemd: Started Session c802968 of user root.
... HH:MM:34 ... systemd: Starting Session c802968 of user root.

{{< /highlight >}}

## 来源分析

关于登录、认证、鉴权，建议完整阅读 鸟哥的Linux私房菜 [第十三章、Linux 帳號管理與 ACL 權限設定](http://linux.vbird.org/linux_basic/0410accountmanager.php#) 不少细节你可能都不知道，比如:

- 系统和登入 UID 的各自取值范围，如何设定一个系统账户(<1000);
- shadow 含密码更改日，锁定天数，更改天数、过期警告/宽限天数、账户失效日;
- 万一 root 密码忘了，咋整？;
- 用户创建文件时属于初始群组还是次要群组，该如何切换有效群组;
- /etc/default/useradd 在不同 Linux 版本如何设定“私有群组”和“公共群组”
- ACL 的 setfacl 和 SUID/SGID/SBIT 权限有何分别？chattr 又是做啥的？
- PAM 模块

改变PAM配置

[How to stop sudo PAM messages in auth.log for a specific user](https://unix.stackexchange.com/questions/224370/how-to-stop-sudo-pam-messages-in-auth-log-for-a-specific-user)
[How PAM works](http://www.tuxradar.com/content/how-pam-works)
[PAM Explanation](http://pig.made-it.com/pam.html)
[The Linux-PAM System Administrators' Guide](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html)
[pam_succeed_if(8) - Linux man page](https://linux.die.net/man/8/pam_succeed_if)

audit 日志的查看方法
[How can I log all process launches in Linux](https://superuser.com/questions/222912/how-can-i-log-all-process-launches-in-linux/)
[How To Use the Linux Auditing System on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-use-the-linux-auditing-system-on-centos-7)
[How To Write Custom System Audit Rules on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-write-custom-system-audit-rules-on-centos-7)


过滤日志的方法

[Logs flooded with systemd messages: Created slice & Starting Session](https://access.redhat.com/solutions/1564823)
[BASIC CONFIGURATION OF RSYSLOG](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-basic_configuration_of_rsyslog.html)

### 日志样例

{{< highlight console >}}

{{< /highlight >}}



参考文档

> -


封面图片来自 [Current paperwork status: doing my taxes…](https://dribbble.com/shots/2082740-Current-paperwork-status-doing-my-taxes) <a href="https://dribbble.com/jan-hendrikholst"><i class="fa fa-dribbble" aria-hidden="true"></i> Jan-Hendrik Holst</a>  

其他图片来源网络
