+++
date = "2020-05-04T01:02:03+08:00"
title = "ARTS 2020w18"
image = "/img/blog/arts-99-per-week/"
draft = true
weight = 2018
tags = ["ARTS"]
+++


<!--more-->

## Algorithm


## Review

## Tip

有时我们会听到负责平台的同事与负责应用开发的同事得如下对话：

开发同事：我这台机器的配置太低了，程序跑不起来，有没有好一点的服务器，换一台
平台同事：这台机器配置还可以了，这多 CPU 和 内存都不够你跑的？你该做程序优化了
开发同事：先找一台 CPU、内存都大一点的，给我试试呗
平台同事：最近机器很紧，我这里不是网吧，你先做优化吧

面对这种性能调优的问题，可能需要从业务特点，应用程序采用的算法，操作系统整体视角，辅助各类测量数据，分析和定位瓶颈，再决定优化方式：软件自身调优，操作系统调，还是要升级硬件。

系统视角、分析定位这些需要技术的积累，非一时一日之功，但安装使用性能指标收集工具，却能很快学习起来，比如红帽系的系统监控软件 Performance Co-Pilot (PCP)，其亮点如下：

- 涵盖 /proc /sys 下所有内容，另外还有针对平台、虚拟、容器、数据库、Java等的丰富插件；
- 规划了统一的测量命名和继承关系、清晰测量的类型、语义、单位等元数据，有助于分析工具的呈现；
- 数据采集和数据回放分析完全分离的架构；
- 采集 API 覆盖各种编程语言，方便对新的测量采集做扩展开发，还拥有 webapi 和 JSON 支持，方便和已有展示应用集成/开发；

下面就按照软件安装部署、查看历史性能指标简单介绍一下。

1. 安装部署软件

由于采集和分析可以分开，待测服务器上做如下安装

{{< highlight plaintext >}}
# RHEL 7.4 以后 和 RHEL8
# yum install pcp-zeroconf
# systemctl enable  pmlogger_daily_report.timer pmlogger_daily_report-poll.timer --now

# RHEL 7 较早版本
# yum install pcp
# systemctl enable pmcd
# systemctl enable pmlogger
# systemctl start pmcd
# systemctl start pmlogger


# firewall-cmd --permanent --zone=public --add-service=pmcd
# firewall-cmd --reload
{{< /highlight>}}

如果希望更密集地进行测量采样（默认间隔60秒），你需要预留更大的硬盘空间——日志存放在`/var/log/pcp`下，然后追加配置参数`-t 10s`、重启服务`systemctl status pmlogger`

{{< highlight plaintext >}}
cat /etc/pcp/pmlogger/control.d/local
#################################
# === LOGGER CONTROL SPECIFICATIONS ===
#Host           P?  S?  directory                               args

# local primary logger
LOCALHOSTNAME   y   n   PCP_LOG_DIR/pmlogger/LOCALHOSTNAME      -r -T24h10m -c config.default -v 100Mb -t 10s
{{< /highlight>}}

分析服务器上需要安装的软件包
{{< highlight plaintext >}}
# yum -y install pcp-system-tools pcp-doc
{{< /highlight >}}


2. 拓展磁盘空间

RHEL 默认使用 XFS 文件系统，调整 `/` 分区的大小要通过 dump/restore 操作完成

确认文件系统的命令 `df -Th`
{{ highlight plaintext }}
# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs   32G     0   32G   0% /dev
tmpfs                   tmpfs      32G   14M   32G   1% /dev/shm
tmpfs                   tmpfs      32G   35M   32G   1% /run
tmpfs                   tmpfs      32G     0   32G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        50G   13G   38G  26% /
/dev/sda1               xfs      1014M  212M  803M  21% /boot
tmpfs                   tmpfs     6.3G   36K  6.3G   1% /run/user/0
/dev/mapper/centos-home xfs       365G   37M  365G   1% /home
{{ /highlight }}

比如希望将 `/home` 的部分空间缩小100G (365 - 100G)，转给 `/` 使其达到 150G (50 + 100G)



如果是 EXT3/4 文件系统，调整 `/` 的大小方法则不同。

3. 查看数据

先从待测服务器打包、压缩日志，然后传到送到分析服务器上。

{{ highlight plaintext }}
# tar czf /tmp/pcp-$(hostname)-data.tgz /var/log/pcp
{{ /highlight }}

在分析服务器上，解压后，文件名前面是每天的时间戳，每天有 index、meta、0、1、xz... 等文件，然后可以通过 `pmdumplog` 查看每天的采集周期
{{ highlight plaintext }}
# cd /var/log/pcp/pmlogger/<myhost>
# for f in $(ls -1 *.0 *.xz | sort -n); do pmdumplog -l $f; done

{{ /highlight }}

日志中还有一个 pmlogger.log 文件，记录了采集服务本身的信息，不同组测量的配置——其采样间隔也可以不同，占用的磁盘空间。

#### 关于合并数据

- 在 pcp 3.11.2-1 版本之前，一个常用操作是合并数天的采集数据，归档到一个文件中再做后续分析。使用 `pmlogextract` 工具。 
- 而在 pcp 3.11.2-1 版本之后，所有工具都支持 `-a /var/log/pcp/pmlogger/<myhost>` ，直接分析跨越多天的日志。 

{{ highlight plaintext }}
# pwd
/var/log/pcp/pmlogger/<myhost>
# pmlogextract $(ls -1 *.[0-9] *.xz | sort -n) /tmp/<myhost>_merge-archive
# pmdumplog -L /tmp/<myhost>_merge-archive

# create a tarball to share with others
# tar czf /tmp/<myhost>_perf.tgz /tmp/merge-archive.{0,meta,index}
{{ /highlight }}

#### 关于查看数据

命令行查看


{{ highlight plaintext }}
# 按照 4 小时间隔，查看合并这几天的 io 平均情况
# pmiostat -a /tmp/<myhost>_merge_perf -t 4h -xt
{{ /highlight }}

图形查看

## Share



封面图片来自 [](d) <a href=""><i class="fa fa-dribbble" aria-hidden="true"></i> </a>