+++
date = "2020-03-02T16:10:29+08:00"
title = "ARTS 2020w09"
image = "/img/blog/arts-90-per-week/"
draft = true
weight = 2009
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

Windows 命令行执行 git 相关操作，中文文件名总是显示为“\数字数字”这种形式，查看非常不方便，排查步骤：

1. 启动 Powershell、cmd、ms terminal 时，默认显示中文字符正常，自建一个含中文名的空文件，通过`dir`命令查看也正常；
2. 在上述窗口，执行 `git ls-files` 查看文件，此时显示不正常，就说明应该查看 git 的相关设置；
3. 查看 git 文档，发现用于 1) 命令配合使用的[`-z`参数](https://git-scm.com/docs/git-config#Documentation/git-config.txt--z) 2) [配置参数`core.quotepath`](https://git-scm.com/docs/git-config#Documentation/git-config.txt-corequotePath)，执行 `git ls-files -z`文件名即显示正常；

扩展阅读：

- Rich (2018-11-15) [Windows Command-Line: Unicode and UTF-8 Output Text Buffer](https://devblogs.microsoft.com/commandline/windows-command-line-unicode-and-utf-8-output-text-buffer/)
- Aupajo [A few of my favourite Git settings](https://gist.github.com/Aupajo/4133515)
- [Git 上手]({{< ref "/blog/setup-git-env.md" >}}) 

## Share


封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>