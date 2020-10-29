+++
date = "2020-08-10T15:07:23+08:00"
title = "ARTS 2020w32"
image = "/img/blog/arts-113-per-week/"
draft = true
weight = 2032
tags = ["ARTS"]
+++


<!--more-->

## Algorithm


## Review

## Tip

git 对每次提交都会生成一个 [40 位的 hash 值](https://git-scm.com/docs/pretty-formats#Documentation/pretty-formats.txt-emhem)。但我们通常引用某一次提交，只需要使用前几位—— [abbreviated tree hash](https://git-scm.com/docs/pretty-formats#Documentation/pretty-formats.txt-emhem) 就够用了。

这个缩略的 hash 值到底留多少位够用？

答案是不一定——[越大的项目需要的位数越长](https://stackoverflow.com/a/21015031)。这个问题本来和我们关系不大，即便不同工具定义了不同默认长度，比如

- gitlab 的 web 界面上显示的缩略 hash 值是 8 位，包括其内建的 CI/CD 的相关变量 `$CI_COMMIT_SHORT_SHA`
- vscode 的 Git Graph 插件显示也为 8 位
- git log 的 pretty 选项中指定 format 时 `%h` 为 7 位

```
git config --global alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short"
```

如果我们通过这个缩略 hash 值查找提交历史，无论 7 位还是 8 位都可以帮助我们定位到对应的提交，但是如果希望使用这个 hash 值作为容器镜像的 tag ，则最好将各个工具统一成同样的长度——比如将 git log 的显示结果从7位拓展到8位，通过下面的指令实现。

```
git config --global core.abbrev 8
```

## Share




封面图片来自 [](d) <a href=""><i class="fa fa-dribbble" aria-hidden="true"></i> </a>