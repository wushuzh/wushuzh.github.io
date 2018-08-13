+++
date = "2017-06-15T20:50:27+08:00"
title = "SVN 数据迁移"
showonlyimage = false
image = "/img/blog/migrate-svn-repo-data/tran-inventory.png"
draft = false
weight = 11
tags = ["svn"]
+++

以下实施过程和命令，亲测有效，供参考
<!--more-->

## SVN 是什么
当然是一个版本管理工具，虽然被 Linus Torvalds 奚落:

> Linus 在某次访谈时提到了对 CVS 的极度厌恶，并提示台下 SVN 的用户也该离开。然后他说  Subversion 是史上最没有意义的项目，SVN(作为 CVS 理念的继任者和替代品)曾经有一段打出的口号是“正确地使用 CVS ”，但这个目标毫无意思，因为以 CVS 的方式管理源代码根本就不可能做对……

但它还是非常很有影响力的。**Apache软件基金会** 就主推 SVN ，当然这也许是因为它本身就是这个基金会旗下的顶级项目。

## 为啥迁移

场景设定： N 年前，刚好有一台淘汰下来的 SPARC 架构的服务器，来做废物再利用，装个操作系统，把 SVN 服务部署到上面。就这么用了几年，但有一天忽然得知 Oracle 要停止对 [OpenSolaris](https://en.wikipedia.org/wiki/Solaris_(operating_system)) 的开发维护，团队考虑到后续维护这些中古硬件和服务的成本，决定要尝试现有服务和数据到当下主流的解决方案上来。

你的选择无非是：

- 准备一个 x86 ，做数据迁移，并以此为主服务器，原有 SVN 服务仅作为备份和镜像、最终淘汰
- 从 SVN 迁移到 Git：那将是一次异构 SCM 的数据迁移

> 如果简单技术尝试，比较快捷的做法也可以利用 Docker

## 新 x86 上的准备

具体的操作步骤可以参看 vultr (2016-08-15) [How to Setup an Apache Subversion (SVN) Server on CentOS 7](https://www.vultr.com/docs/how-to-setup-an-apache-subversion-svn-server-on-centos-7)，这里只提一下基本步骤，不重复细节。值得一提的是，这篇文章来自搬瓦工，现在各个 VPS 提供商(还有 Digital Ocean )都维护了一大批很不错的各种服务的初始安装配置文档，文档质量都很不错。

1. 安装一个 CentOS 7 的最小集，配好yum源，把所有已安装的软件包更新到最新版，重启
2. 安装配置 Apache ，如移除欢迎页，禁掉 /var/www/html 等
3. 安装 SVN 和相应 Apache 模块
4. 加载 SVN 模块，并做身份认证配置
5. 创建 SVN 用户密码文件，并严格限定读取权限
6. 设定 SVN 各个用户权限：管理员、对某仓库可读写、对某仓库只可读
7. 设定防火墙，并启动 Apache 服务
8. 打开浏览器，访问 http://ip/svn/repo 用相应用户密码测试访问

其他，比如配置 CGI 使得可以通过网页自行更改密码，或添加删除用户，参见 Erik Mugele (2005-11) [Tools to Manage Subversion Users and Passwords](http://www.teuton.org/~ejm/svnpasswd/)

## 建议和常见问题

{{< highlight console >}}
# 日志消息 1：
  Failed to load the AuthzSVNAccessFile:
      Can't open file '/svn/authz': Permission denied

# 日志消息 2：
  Can't open file '/svn/repo1/format': Permission denied

# 日志消息 3：
  Could not open the requested SVN filesystem

# 解决方式：设定相关文件的 SElinux security context
chcon -Rv --type=httpd_sys_content_t /SVN_ROOT_PATH
{{< /highlight >}}

- 身份验证时和浏览仓库时，浏览器提示无法访问，服务器端httpd的error日志中显示无权限。
- 我查看日志时发现系统的时间不对，这种对外提供服务的机器最好还是做一下 ntp 时间同步，不然一边查问题看日志，还要一边做时间换算就很麻烦了。
参见
  - Matei C. (2015-03-14) [Setting Up “NTP (Network Time Protocol) Server” in RHEL/CentOS 7](https://www.tecmint.com/install-ntp-server-in-centos/))
  - Aaron K [How to Set Time, Timezone and Synchronize System Clock Using timedatectl Command](https://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/)  
- 后续的数据备份，转移，导入的操作可能花费很长时间，因此建议所有操作都进入 tmux 或  screen 执行，除非你能保证绝对稳定的网络连接，比如直连服务器而不是远程登录

## 数据迁移方案

A. 使用 dump 选择需要的数据做备份，并在另一台服务器上做数据复原  
B. 使用 svnsync 在另外一台机器上生成当前数据的只读镜像

方案 A dump ：

1. 原服务器上暂停服务，做仓库备份
2. 新服务器上创建仓库，导入备份
3. 更改用户访问权限文件 authz , 为新建的仓库添加用户组（只给只读权限）
4. 浏览器上测试访问，看是否有 SELinux 的问题等
5. 两台服务器上执行 svnlook youngest REPO_PATH 结果应该是一样

{{< highlight bash >}}
svnlook history REPO_PATH --show-ids --limit 10
svnlook info -r SOME_REVISION_ID
svnlook info REPO_PATH # 查看仓库最新一次提交
{{< /highlight >}}  

> 如果是 svn client，则使用```svn info```查看远端中央仓库的 URL 以及最后当下仓库的最后更新时间。

方案 B svnsync

1. 创建一个用于镜像的仓库
2. 之后通过在新服务器上设定钩子，仅允许特定用户执行镜像的操作
3. 同步的初始化设置，哪个为源仓库，哪个为镜像仓库 svnsync initialize ...
4. 同步进行中 svnsync synchronize ...
5. 在源服务器上设定钩子，实现非交互式的自动同步

这个方法也可以用于增量备份:
{{< highlight bash >}}
svnsync initialize --allow-non-empty file:///MIRROR_PATH SOURCE_URL
svnsync sync file:///MIRROR_PATH
{{< /highlight >}}

但它也有一个小缺陷，即对于已同步过的历史的一些属性不会再同步，需要通过 svnsync 子命令 copy-revprops 显式完成。

## 尾声

数据同步了，剩下的就是把原有的用户身份认证文件拷贝到对端机器上，确认账户仍可使用。之后并重置(清空)原有机器的鉴权文件，迫使团队成员只能将代码提交到新的仓库地址。再之后，别忘了更改持续集成部分的仓库地址。

## 相关命令

---

### Linux

{{< highlight bash >}}
sudo yum update
sudo shutdown -r now
sudo yum install httpd
sudo sed -i 's/^/#&/g' /etc/httpd/conf.d/welcome.conf
sudo sed -i \
  "s/Options Indexes FollowSymLinks/Options FollowSymLinks/" \
  /etc/httpd/conf/httpd.conf
{{< /highlight >}}

{{< highlight bash >}}
sudo yum install subversion mod_dav_svn
sudo vi /etc/httpd/conf.modules.d/10-subversion.conf
{{< /highlight >}}

{{< highlight bash >}}
sudo mkdir /svn
cd /svn
sudo svnadmin create repo1
sudo chown -R apache:apache repo1
{{< /highlight >}}

{{< highlight bash >}}
sudo mkdir /etc/svn
sudo htpasswd -cm /etc/svn/svn-auth user001
sudo chown root:apache /etc/svn/svn-auth
sudo chmod 640 /etc/svn/svn-auth
sudo htpasswd -m /etc/svn/svn-auth user002
sudo htpasswd -m /etc/svn/svn-auth user003
sudo cp /svn/repo1/conf/authz /svn/authz
sudo vi /svn/authz
{{< /highlight >}}

{{< highlight bash >}}
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
{{< /highlight >}}

---

### Solaris

查看某个文件从属的软件包
{{< highlight bash >}}
pkgchk -lp /usr/local/bin/svnserve
{{< /highlight >}}

查看软件包中所有内容(文件)
{{< highlight bash >}}
pkgchk -l SMCsvn1612
pkgchk -l SMCap2221
{{< /highlight >}}

查看已经系统中所有已安装的软件包
{{< highlight bash >}}
pkginfo
{{< /highlight >}}


缩略语解释

> - [SPARC](https://en.wikipedia.org/wiki/SPARC): 精简指令 RISC 集架构；和 x86 复杂指令集 CISC 相对
> - SCM: Software configuration management 软件配置管理
> - CVS: Concurrent Versions System 一个上个世纪的版本管理工具，据说最初是用一堆 shell 脚本实现的。
> - ASF：Apache软件基金会，美国的非盈利组织

参考文档

> - Online Subversion free book [Version Control with Subversion](http://svnbook.red-bean.com/)
> - Stackoverflow [centos apache svn forbidden related to selinux](https://stackoverflow.com/a/40891894)
> - Geoffrey H. (2015-09-23) [How to use svnsync to create a mirror backup of your Subversion repository](http://www.cardinalpath.com/how-to-use-svnsync-to-create-a-mirror-backup-of-your-subversion-repository/)
> - [Mirror a Subversion repository](http://www.microhowto.info/howto/mirror_a_subversion_repository.html)
> - 陈浩 (2010-11-17) [版本管理器的发展史](http://coolshell.cn/articles/3288.html)

封面图片来自 [Transferring Inventory](https://dribbble.com/shots/2764840-Transferring-Inventory) <a href="https://dribbble.com/Coleman811"><i class="fa fa-dribbble" aria-hidden="true"></i> Rye</a>  

其他图片来源网络
