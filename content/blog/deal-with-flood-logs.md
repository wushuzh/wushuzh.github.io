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

["syslog 标准及其实现"]({{< ref "/blog/syslog-std-in-general.md" >}}) 中最后提到了一种 syslog 的实现:“快火箭日志”。本篇就针对它说说遇到“洪水报文消息”的处理方法。

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
- PAM 模块简介: 配置和控制流程

### Linux PAM

Linux 作为服务器操作系统，必然同时运行着多种服务——但如果让每个服务定义各自的账户、密码和验证机制无疑是一种浪费。于是 PAM (可嵌入认证模块)就被设计成一种提供通用的、可定制需求的认证接口。开发者只要定义设定其服务的认证需求，PAM 就按相应流程结合登录时的实际情况向服务和应用返回认证结果。

PAM 的设定配置文件都存放于 /etc/pam.d/ ，配置文件一般三列

第一列是验证类别 type（设定顺序的一般也同下）:

1. 身份认证/auth
2. 账户授权/account
3. 登入登出/session
4. 密码变更/password

第二列是控制标志 flag

- required 无论当前判定结果如何，都会进行后续步骤
- requisite 一旦当前判定失败，立刻终止，不进行后续步骤
- sufficient 一旦当前判定陈宫，立刻终止，不进行后续步骤
- optional 一般用于日志记录

{{< figure src="/img/blog/deal-with-flood-logs/pam-run_stack.png" title="ctrl flow from Oracle" >}}

可以断定的是，之前 message 和 security 中看到的大量 session 切换的日志，就源自 PAM 验证中的日志记录。

我按照 [How to stop sudo PAM messages in auth.log for a specific user](https://unix.stackexchange.com/a/224444) 中的解决方案，发现借助 [pam_succeed_if(8)](https://linux.die.net/man/8/pam_succeed_if) 修改 su、su-l 等配置也确实可以屏蔽掉一部分日志。但直接修改系统预设定的 pam 配置文件是很有风险的。如果对整个验证流程理解不透彻，则可能造成

- 验证逻辑错误，原本该被判定合法的认证授权被误判为非法
- 验证逻辑漏洞，非法登录被忽略记录，乃至放行，给系统引入安全审计漏洞

## rsyslog

除了上述更改鉴权流程本身，更为保险的方法是更改输出日志的配置。显式地制定可以忽略的日志规则，存入配置文件，例如 /etc/rsyslog.d/ignore-su-session-slice.conf，然后重启服务``` systemctl restart rsyslog```

{{< highlight console >}}
if ($programname == "su" and ($msg contains "root on none"))
  or ($programname == "systemd"
      and ($msg contains "Starting Session"
           or $msg contains "Started Session"))
  or ($programname == "systemd-logind"
      and ($msg contains "New session"
           or $msg contains "Removed session")))
then stop
{{< /highlight >}}

参考文档

> - Wikipedia [Pluggable authentication module](https://en.wikipedia.org/wiki/Pluggable_authentication_module) and [Linux PAM](https://en.wikipedia.org/wiki/Linux_PAM)
> - Serverfault [Understand PAM and NSS](https://serverfault.com/a/538503)
> - Vishal Srivastava (2009-03-10) [Understanding and configuring PAM](https://www.ibm.com/developerworks/library/l-pam/index.html)
> - [How PAM works](http://www.tuxradar.com/content/how-pam-works)
> - [PAM Explanation](http://pig.made-it.com/pam.html)
> - Andrew G. Morgan and Thorsten Kukuk [The Linux-PAM System Administrators' Guide](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html)
> - [Rsyslog conf](http://www.rsyslog.com/doc/v8-stable/configuration/index.html)

封面图片来自 [Current paperwork status: doing my taxes…](https://dribbble.com/shots/2082740-Current-paperwork-status-doing-my-taxes) <a href="https://dribbble.com/jan-hendrikholst"><i class="fa fa-dribbble" aria-hidden="true"></i> Jan-Hendrik Holst</a>  

其他图片来源网络
