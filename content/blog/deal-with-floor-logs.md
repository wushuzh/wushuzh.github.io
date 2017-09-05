+++
date = "2017-09-05T22:50:33+08:00"
title = "重复日志，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-flood-logs/taxes.png"
draft = true
weight = 47
+++

如何过滤无用的系统日志……
<!--more-->

系统日志被大量重复信息充斥，害得正常日志偶现其中，特别不限眼，严重影响了系统查看和安全审计

鸟哥的Linux私房菜 [第十三章、Linux 帳號管理與 ACL 權限設定](http://linux.vbird.org/linux_basic/0410accountmanager.php#)

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
