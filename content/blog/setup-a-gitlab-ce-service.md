+++
date = "2017-06-18T21:18:51+08:00"
title = "GitLab CE 安装"
showonlyimage = false
image = "/img/blog/setup-a-gitlab-ce-service/lab_notebook.png"
draft = false
weight = 13
tags = ["git"]
+++

如何搭建 GitLab 以获得类似 GitHub 的服务
<!--more-->

## Git 简介
作为 Linus Torvalds 的另一个伟大发明，[Git](https://en.wikipedia.org/wiki/Git) 应该已经成为当下最流行的源代码管理工具。它甚至被定位成一般意义上追踪文档变更或多人协同编辑的工具了。

使用 Git 最大开源社区是 Linux [内核项目](https://en.wikipedia.org/wiki/Linux_kernel)，向它贡献代码的开发者的人数高达 12000 人，来自全球 1200 家公司。另一个前些日子很热的新闻是微软将 Windows 开发迁移到 Git 上，并对其做了一些定制 (如 GVFS )。其他的 Git 应用案例包括

- 应用 git-lfs 保存较大的二进制文件;
- [查看](https://github.com/blog/1465-stl-file-viewing)、[比较](https://github.com/blog/1633-3d-file-diffs) CAD文件的变更历史 ( [STL](https://en.wikipedia.org/wiki/STL_(file_format)) 格式保存 )；
- 管理法案、法律文件的[探索](https://www.quora.com/Could-Git-be-used-to-track-bills-in-Congress);

所有的主流源代码仓库服务提供商都支持 Git。最大最知名的当属被称为编程交友平台的 GitHub (吉祥物为一只章鱼猫 )，和它类似还有 SourceForge，Bitbucket，GitLab，CodePlex, Launchpad （排名按 2017 六月 Alexa 排名）。

<img alt="GitHub Octocat" src="/img/blog/setup-a-gitlab-ce-service/ironcat.png" class="img-responsive">

## GitLab 简介
[GitLab](https://en.wikipedia.org/wiki/GitLab) 是一个基于网页的开源 Git 仓库管理器，附送代码审核、缺陷跟踪、Wiki、编译构建等附加功能。作为少数的可以自行搭建相应服务的网站，一般的开发者只要有个VPS就可以很快搭建起来。GitHub 作为付费产品，当然也有所有这些服务，但通常你要注册为其付费会员(每月7美元)

关于 GitLab 的新特性，其官网上有[介绍视频](https://youtu.be/PoBaY_rqeKA)：作者在短短十分钟内，演示了利用 GitLab 9 完成如下事项

1. 创建一个工作组，在其下创建(导入)一个实例项目
2. 设定基于 Mattermost (一个可自行搭建的类似 slack 的聊天协作工具)的自动集成部署
3. 在 Mattermost 内和组员讨论后，形成更改项目的新决议
4. 在 Mattermost 内通过 GitLab 机器人直接创建一个 issue
5. 从 Mattermost 导航到项目 issue 页，通过看板，将此 issue 从 Todo 转到 Doing 状态
6. 通过 GitLab 的内置终端，直接访问部署环境
7. 通过 GitLab 的网页端编辑器，创建新分支，并将新的提交合并到主分支（并关闭 issue， 删除临时分支）
8. GitLab 自动触发自动部署，在其他团队成员审阅代码前，就开始了新代码的部署，这样审核代码时，就可以查看部署是否成功，并导航到新部署容器上查看改成后的界面……
9. 当分支真正被接受合并后，又一次触发了主分支上的自动集成，自动部署到待命区服务器 (staging server)
10. 之后从 Mattermost 通过 Bot 触发从待命区部署到生产环境，如果生产环境是集群，这里还可以看到当前部署到第几个实例了
11. 另外，你还可以直接查看部署服务器的负荷的曲线图，回看总结本次“从产生想法到做最终部署”各阶段所用的时间

{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-pages.png" title="GitLab Pages" >}}
{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-issues.png" title="GitLab Issues" >}}
{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-issue-board.png" title="GitLab Issues Board" >}}
{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-merge-conflicts.png" title="GitLab Merge" >}}
{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-review-apps.png" title="GitLab Review" >}}
{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-pipelines.png" title="GitLab Pipeline" >}}
{{< figure src="/img/blog/setup-a-gitlab-ce-service/gitlab-cycle-analytics.png" title="GitLab Analysis" >}}

这么梦幻的一个工作流，一分钱不用花自己折腾一下就可以拥有，心动就参考下文开始行动吧

## 安装过程
网上普遍都是[基于 Ubuntu 的安装指南](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-gitlab-on-ubuntu-16-04)，那就按照最普遍的系统进行实践：

{{< figure src="/img/blog/setup-a-gitlab-ce-service/ubuntu-16-04-xenial-xerus-squirrel.jpg" title="Ubuntu 1604 LTS" >}}

0. 所有步骤都建议再 tmux 中执行，以免网络抖动后悲剧
1. Ubuntu 推荐版本 16.04 LTS Xerus
2. 配置 GitLab deb 源并安装最新安装包
3. 执行 GitLab 初始配置
4. 设置防火墙对 SSH 和 HTTP 链接开放
5. 登录网页，设定密码，管理员身份设定，是否全部或对某域名邮箱开发注册等等
6. 如果你有自己的域名，通过 Let's Encrypt 设定 HTTPS(https://www.digitalocean.com/community/tutorials/how-to-secure-gitlab-with-let-s-encrypt-on-ubuntu-16-04)
7. Enjoy

我自己提交代码都用 SSH，就没搞 HTTPS

<img alt="Welcome to GitLab" src="/img/blog/setup-a-gitlab-ce-service/fox-snow.gif" class="img-responsive">

缩略语解释

> - GVFS：Git Virtual File System
> - Xerus: 仅存于非洲的地松鼠亚科，注意不要和南非狐獴弄混，前者属于啮齿目，后者属于食肉目，除了昆虫，有时候会吃蜥蜴，蛇，蝎子什么的
> - Mattermost: 一个同样可以自行搭建的类似 slack 提供团队协作的云交流工具，但新增了不少企业级的特色功能，比如查看诸如关于配置变更，登录，用户行为等各类服务器日志，数据的合规访问，各式CI和CD集成操作。其客户有 Redhat、Intel、Mozilla、Sumsung 等等

参考文档

> - Justin E.(2016-10-14). [How To Install and Configure GitLab on Ubuntu 16.04.]( https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-gitlab-on-ubuntu-16-04)
> - Mitchell A.(2016-04-21) [initial-server-setup-with-ubuntu-16.04]( https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
> - https://explainxkcd.com/wiki/index.php/1597:_Git

文中图片来源网络
GitLab 吉祥物是一个抽象的 “貉”/“狸”脸，但我觉的如果能请来 Megan Fox 为其代言，一定会大获成功
{{< figure src="/img/blog/setup-a-gitlab-ce-service/megan-fox.jpg" title="Megan Fox" >}}
