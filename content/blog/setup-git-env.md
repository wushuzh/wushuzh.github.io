+++
date = "2017-11-08T20:11:49+08:00"
title = "Git 上手"
showonlyimage = false
image = "/img/blog/setup-git-env/kimonotocat.png"
draft = false
weight = 15
+++

简单介绍 git 入門向导和协作流程
<!--more-->

### 准备工作

[Git Immersion](http://gitimmersion.com/) 的设定是编写 ruby 脚本的场景，类似的还有 [Git Howto](https://githowto.com) 。无论操作什么文件，这些教程的本质是相同的，都是帮初学者尽快掌握 git 日程操作。直觉是一个工具 20% 的用法有着极为高频的使用率，先把他们掌握从而尽快入門。而随着操作的日趋熟练，再慢慢地去探索更高阶的操作。

> Immersion 的好处是它提供了教程本身的 html 文件，这使得就算是没有网络，你可以本地调出内容目录，随时继续/重温相关内容。

用 curl 下载时请注意，有时访问原始连接得到 3xx 相应，为了让 curl 继续访问更新的地址，需要加上 -L 选项，仅用 -O 下载得到的文件当然是不对的。 

{{< highlight console >}}
$ curl -O http://githowto.com/git_tutorial.zip
$ unzip git-tutorial.zip
Archive:  git_tutorial.zip
  End-of-central-directory signature not found.
  Either this file is not a zipfile, or it constitutes
  one disk of a multi-part archive.
  In the latter case the central directory
  and zipfile comment will be found
  on the last disk(s) of this archive.

unzip: cannot find zipfile directory in
  one of git_tutorial.zip or git_tutorial.zip.zip,
  and cannot find git_tutorial.zip.ZIP, period.

$ curl -LOk http://githowto.com/git_tutorial.zip
$ unzip git-tutorial.zip
...


$ cd git_immersion/git_tutorial/html/index.html && \
  google-chrome-stable index.html

{{< /highlight >}}

### 基本操作

- 1-2 设置全局用户名和邮箱，根据 OS 做 CRLF 相关设置
- 3-4 将项目加入版本库，跟踪、提交到版本库，查询状态 add
- 5-9 更新文件，区分提交、已暂存和未暂存三个状态: git 关注变化而非文件 commit、status
- 10-11 各种花哨的查看历史的方法，定义常用命令别名 log、alias
- 12 回退到某一历史版本，返回到最新版本 checkout
- 13 为某一版本打标签，查看标签、回退到某标签的版本 tag
- 14-15 放弃未暂存/已暂存的更改 checkout、reset
- 16-18 放弃已提交的更改的方法和风险 revert、reset
- 19 立刻更新最新提交版本 commit --amend
- 20-21 移动、改名操作 mv
- 22-23 窥视 .git 内部
- 24-27 创建切换分支 checkout -b、查看所有分支 -all
- 28-30 合并(冲突)分支 merge
- 31-35 另一种合并方式 rebase
- 36-40 克隆版本库、远端库/分支 clone、remote -a、branch -a
- 41-44 获取、合并远端的新增提交 fetch and merge、pull
- 45 本地追踪远端分支 branch --track
- 46-50 原始仓库 --bare、remote add、push、pull --track、daemon

### 进阶操作

有时候你发现新建一个 git 仓库，有了几次 commit 但是 author 和 email 却错了。这时可以使用 git-filter-branch 可以重写之前的提交历史，但之前所有提交的 SHA1 也都会变化，所以更使用于本地修改尚未提交到远端的情况。 详见 [change author / commiter details in all commits (WARNING: Will change all SHA1s)](https://gist.github.com/ecentinela/199670/7fdb39cbfc2890820c8e8ef64e1184716a24f1cc)

> What is the difference between author and committer in Git? [1](https://stackoverflow.com/a/6755848) and [2](https://stackoverflow.com/a/18754896)

此指令还会对修改前的历史做相应备份，方便你修改后发现问题的回滚。详见 [Undo git filter-branch](https://stackoverflow.com/a/27975288)

参考文档

> - [Pro Git 中文版] 2014 (https://git-scm.com/book/zh/v2)
> - answer [Difference between CR LF, LF and CR line break types?](https://stackoverflow.com/a/1552775/4393386)
> - Wikipedia [Newline](https://en.wikipedia.org/wiki/Newline)

封面图片来自 [Kimonotocat](https://octodex.github.com/Kimonotocat) <a href="https://github.com/jeejkang"><i class="fa fa-github" aria-hidden="true"></i> jeejkang</a>
