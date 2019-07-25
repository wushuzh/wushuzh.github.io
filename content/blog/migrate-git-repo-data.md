+++
date = "2017-06-20T22:17:17+08:00"
title = "GitLab 数据迁移"
showonlyimage = false
image = "/img/blog/migrate-git-repo-data/tran-between-pc.png"
topImage = "/img/blog/migrate-git-repo-data/tran-between-pc.gif"
draft = false
weight = 14
+++

迁移过程同样亲测有效，供参考
<!--more-->

之前从 digital ocean 申请的 VPS ，网络代理的使用感受是无论是使用日本还是新加坡的数据中心，网络代理的质量都非常差。

记得之前有人提到过 linode 从联通北京到日本的网络质量更有保证，之前 gitlab 数据备份时，经常因为内存不够，导致备份失败（需要重启各种重试），而这次比较了两家相同价位的机器，linode 居然内存更多，终于坚定了我迁移的决心。

{{< highlight txt >}}
root@mydo:/tmp# gitlab-rake gitlab:backup:create
rake aborted!
Errno::ENOMEM: Cannot allocate memory - whoami
/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/popen.rb:23:in `popen'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/task_helpers.rake:73:in `run'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/task_helpers.rake:94:in `warn_user_is_not_gitlab'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/backup.rake:8:in `block (3 levels) in <top (required)>'
Tasks: TOP => gitlab:backup:create
(See full trace by running task with --trace)
{{< /highlight >}}

VPS 自己搭建的 gitlab ，上面又一些自己创建的 repo ，是迁移唯一需要处理的数据。

操作系统用的 Ubuntu 14.04.4 LTS ，gitlab 8.3.2 搭建好后也没有做过大版本升级。这次打算把 gitlab 升级和数据迁移一起做了。

大概流程：

1. 备份 gitlab 配置文件 `sudo sh -c 'umask 0077; tar -cf $(date "+etc-gitlab-%s.tar") -C / etc/gitlab'`；
2. 反复启动 VM，启停 gitlab `sudo gitlab-ctl stop/start`（因为内存不够，不重启直接备份经常失败），备份代码仓库直到成功；
3. 确认升级路径，因为长时间没有升级，gitlab 已经版本号已经到达 9，需要两步升级；
    - 升级 gitlab 到 8.x.x 上的最后一个版本；
    - 升级 gitlab 到 9.x.x ；
4. 在 linode 上建立一个实例，操作系统使用新版本的 LTS 16.xx ，安装 gitlab 9.x.x ，在上面恢复之前的备份数据；

将第三步的一些操作日志截取如下：

{{< highlight shell >}}
apt-mark hold gitlab-ce

sudo apt-get install gitlab-ce

# The following packages will be upgraded:
#   gitlab-ce
# 1 upgraded


# gitlab preinstall: Automatically backing up only the GitLab SQL database (excluding everything else!)
# Dumping database ...
# Dumping PostgreSQL database gitlabhq_production ... [DONE]
# Creating backup archive: 1497655521_gitlab_backup.tar ... done

# Unpacking gitlab-ce (9.2.6-ce.0) over (8.3.2-ce.0) ...
# Setting up gitlab-ce (9.2.6-ce.0) ...
# Installing Cookbook Gems:
# Compiling Cookbooks...

# Shutting down all GitLab services except those needed for migrations
# ok: down: gitlab-workhorse: 0s, normally up
# ok: down: logrotate: 1s, normally up
# ok: down: nginx: 0s, normally up
# ok: down: postgresql: 1s, normally up
# ok: down: redis: 0s, normally up
# ok: down: sidekiq: 0s, normally up
# ok: down: unicorn: 1s, normally up
# ok: run: postgresql: (pid 9117) 0s
# ok: run: redis: (pid 9119) 0s
# run: postgresql: (pid 9117) 0s; run: log: (pid 1031) 725215s
# run: redis: (pid 9119) 0s; run: log: (pid 1027) 725215s
# Reconfiguring GitLab to apply migrations

# 中间应该是 chef 控制
#   比如将 gitlab 9 中新增的配置合并到已有的配置文件中
#   猜测这里的更新脚本都是从某个版本到某个版本固定的，
#   比如 从 8.1.2 -> 8.2.3 -> 8.3.4 ... -> 8.9.9
#   

# Recipe: gitlab::postgresql-bin
# Recipe: gitlab::default
# Recipe: gitlab::web-server
# Recipe: gitlab::users
# Recipe: gitlab::gitlab-shell
# Recipe: gitlab::gitlab-rails
# Recipe: gitlab::add_trusted_certs
# Recipe: gitlab::default
# Recipe: runit::upstart
# Recipe: gitlab::redis
# 
# Recipe: gitlab::postgresql_user
# Recipe: gitlab::postgresql
# Recipe: gitlab::postgresql-bin
# Recipe: gitlab::database_migrations
# 
# Recipe: gitlab::gitlab-rails
# Recipe: gitlab::logrotate_folders_and_configs
# Recipe: gitlab::unicorn
# Recipe: gitlab::sidekiq
# Recipe: gitlab::gitaly
# Recipe: gitlab::gitlab-workhorse
# Recipe: gitlab::mailroom_disable
# Recipe: gitlab::nginx
# Recipe: gitlab::remote-syslog_disable
# Recipe: gitlab::logrotate
# Recipe: gitlab::mattermost_disable
# Recipe: gitlab::gitlab-pages_disable
# Recipe: gitlab::registry_disable
# Recipe: gitlab::gitlab-healthcheck
# Recipe: gitlab::prometheus_user
# Recipe: gitlab::prometheus
# Recipe: gitlab::node-exporter
# Recipe: gitlab::redis-exporter
# Recipe: gitlab::postgres-exporter
# Recipe: gitlab::gitlab-monitor
# Recipe: gitlab::default
# Recipe: gitlab::gitlab-rails
# Recipe: gitlab::gitaly
# Recipe: gitlab::gitlab-workhorse

# Chef Client finished, 8/394 resources updated in 12 seconds
# Running reconfigure: OK
# Database upgrade is complete, running analyze_new_cluster.sh
# ==== Upgrade has completed ====
# Please verify everything is working and run the following if so
# rm -rf /var/opt/gitlab/postgresql/data.9.2.18
# Toggling deploy page:rm -f /opt/gitlab/embedded/service/gitlab-rails/public/index.html
# Toggling deploy page: OK
# Toggling services:ok: run: gitaly: (pid 10722) 0s
# ok: run: gitlab-monitor: (pid 10727) 0s
# ok: run: logrotate: (pid 10729) 0s
# ok: run: node-exporter: (pid 10735) 0s
# ok: run: postgres-exporter: (pid 10740) 0s
# ok: run: prometheus: (pid 10744) 0s
# ok: run: redis-exporter: (pid 10748) 0s
# ok: run: sidekiq: (pid 10752) 0s
# Toggling services: OK
# Ensuring PostgreSQL is updated: OK
# Restarting previously running GitLab services
# ok: run: gitlab-workhorse: (pid 9883) 62s
# ok: run: logrotate: (pid 10729) 1s
# ok: run: nginx: (pid 10757) 0s
# ok: run: postgresql: (pid 10759) 0s
# ok: run: redis: (pid 9629) 187s
# ok: run: sidekiq: (pid 10752) 1s
# ok: run: unicorn: (pid 10766) 0s
# 
# Upgrade complete! If your GitLab server is misbehaving try running
# 
#    sudo gitlab-ctl restart
# 
# before anything else. If you need to roll back to the previous version you can
# use the database backup made during the upgrade (scroll up for the filename).


gitlab-rake gitlab:env:info

# System information
# System:         Ubuntu 14.04
# Current User:   git
# Using RVM:      no
# Ruby Version:   2.3.3p222
# Gem Version:    2.6.6
# Bundler Version:1.13.7
# Rake Version:   10.5.0
# Redis Version:  3.2.5
# Git Version:    2.11.1
# Sidekiq Version:5.0.0
# 
# GitLab information
# Version:        9.2.6
# Revision:       332a71d
# Directory:      /opt/gitlab/embedded/service/gitlab-rails
# DB Adapter:     postgresql
# URL:            http://mydo
# HTTP Clone URL: http://mydo/some-group/some-project.git
# SSH Clone URL:  git@mydo:some-group/some-project.git
# Using LDAP:     no
# Using Omniauth: no
# 
# GitLab Shell
# Version:        5.0.4
# Repository storage paths:
# - default:      /var/opt/gitlab/git-data/repositories
# Hooks:          /opt/gitlab/embedded/service/gitlab-shell/hooks
# Git:            /opt/gitlab/embedded/bin/git


sudo gitlab-rake gitlab:backup:create
{{< /highlight >}}

关于如何升级 Ubuntu 操作系统:

[Update the system from terminal](https://askubuntu.com/questions/462449/update-the-system-from-terminal)

{{< highlight shell >}}
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
{{< /highlight >}}

我并没有做这一步，直接在 linode 上申请自带特定 OS 版本的 image 启动实例即可。

封面图片来自 [License Transfer](https://dribbble.com/shots/2617161-License-Transfer) <a href="https://dribbble.com/zacdixon"><i class="fa fa-dribbble" aria-hidden="true"></i> Zac Dixon</a>
