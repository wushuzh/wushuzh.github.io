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

本篇作为  “[SVN 数据迁移]({{< relref "blog/migrate-svn-repo-data.md" >}})” 的续篇，讨论将 svn 代码仓库的异构迁移：即从 SVN 全面转到 Git

## 主要步骤

1. 为迁移准备本地环境
2. 将 SVN 仓库转成一个本地 Git 仓库
3. 将 SVN 的后续更新不断地同步到这个本地 Git 仓库
4. 将 Git 仓库共享给所有开发人员
5. 后续的更改都指向新 Git 仓库

其中前三个步骤比较关键，依次是“准备” -> “转换” -> “同步”。作用是将 SVN 所有提交历史都导入 Git 仓库里。

> **最佳实践**：为团队指定一个迁移负责人(Migration lead)，此三步都在此人本地电脑上完成。

---

前三步完成后，迁移负责人的本地 Git 仓库，已经和能和远端 SVN 仓库保持同步。之后迁移负责人可以将新 Git 仓库上传到远端 GitWeb，GitLab，Gerrit，GitHub，Bitbucket 等等(第四步)，供团队成员自行克隆，查看历史、熟悉 Git 命令，最后别忘了对集成编译做相应更改(第五步)。

> **最佳实践**:
>
> 1. 在整个团队w未确认可切换到 Git 工作流前，一定保持一条单一从 SVN 到 Git 的同步通路——避免乱套——比如，团队成员对新的远端 Git 仓库只有读权限；仅由迁移负责人在这一特殊历史时期做代码同步。
> 2. 当整个团队确认对 Git 迁移完全准备好了的时候，又千万记得冻结原 SVN 仓库，从此时间点后，所有的新代码提交都使用 Git。

## 操作细节

提前准备一个 Linux 环境：

> Linux 相比 Windows 的 NTFS 和 OS X 环境的优势在于对文件名的大小写敏感

首先上面安装好 git 、jre 、tmux，并从 Bitbucket 下载一个 Atlassian 维护用于迁移的 jar 文件。

{{< highlight bash >}}
java -jar ~/svn-migration-scripts.jar verify
java -jar ~/svn-migration-scripts.jar authors \
    <svn-repo>/<project> > authors.txt
# edit authors.txt with git account and email addr

git --version
git version 2.13.3

# due to svn-migration-scripts.jar only work for git versions v1.x
#    you need to add options '--prefix=""' with git svn clone cmd
#    otherwise you cannot generate local branches in later step

git svn clone --prefix="" --stdlayout --authors-file=authors.txt
    <svn-repo>/<project> <git-repo-name>

cd <git-repo-name>
git branch -r
# dry run
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git
# for real
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git --force
git branch

git svn fetch
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar sync-rebase
java -Dfile.encoding=utf-8 -jar ~/svn-migration-scripts.jar clean-git --force

{{< /highlight >}}

参考文档

> - Atlassian [Migrate to Git from SVN](https://www.atlassian.com/git/tutorials/migrating-overview)
> - svn-migration-scripts [issues # 11](https://bitbucket.org/atlassian/svn-migration-scripts/issues/11/clean-git-creates-local-branches-only-to) clean-git creates local branches only to delete them again -- no change in the end
