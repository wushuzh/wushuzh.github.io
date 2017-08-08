+++
date = "2017-08-06T16:12:21+08:00"
title = "从 SVN 迁移到 Git"
showonlyimage = false
image = "/img/blog/migrate-svn-to-git/upg-2-git.png"
topImage = "/img/blog/migrate-svn-to-git/char-upd.gif"
draft = false
weight = 12
+++

以下实施过程和命令，亲测有效，供参考
<!--more-->

# 将代码从 SVN 迁移到 Git

本篇作为 SVN 服务和数据迁移的续篇，讨论了已有代码仓库的异构迁移：即从 SVN 全面转到 Git

## 主要步骤
1. 为迁移准备本地环境
2. 将 SVN 仓库转成一个本地 Git 仓库
3. 将 SVN 的后续更新不断地同步到这个本地 Git 仓库
4. 将 Git 仓库共享给所有开发人员
5. 后续的更改都指向新 Git 仓库

其中前三个步骤比较关键，依次是“准备” -> “转换” -> “同步”。作用是将 SVN 所有提交历史都导入 Git 仓库里。

> **最佳实践**：为团队指定一个迁移负责人(Migration lead)，此三步都在此人本地电脑上完成。

---
前三步完成后，迁移负责人的本地 Git 仓库，和能和对应的 SVN 仓库最新版本同步。之后迁移负责人可以将新 Git 仓库上传到，供团队成员自行克隆，查看历史、熟悉 Git 命令，并对集成编译做相应更改。

> **最佳实践**: 在团队确认可以切换到 Git 工作流之前，保持唯一的从 SVN 到 Git 的同步——避免乱套——团队成员对 Git 仓库只有只读权限。仅由迁移负责人在这一特殊历史时期做代码同步。最终当团队对 Git 迁移完全准备好了的时候，再冻结原 SVN 仓库，从此时间点后，所有的新代码提交都使用 Git。

参考文档
> - https://www.atlassian.com/git/tutorials/migrating-overview
> - https://www.atlassian.com/git/tutorials/learn-git-with-bitbucket-cloud
