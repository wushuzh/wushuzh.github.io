+++
date = "2020-01-27T11:27:38+08:00"
title = "ARTS 2020w04"
image = "/img/blog/arts-85-per-week/"
draft = true
weight = 2004
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

## Share

在 Spring Tool Suite 上创建新的 Spring 项目的步骤：

1. choco 上的 STS 已经过期很久，不要用，直接去官网下载基于 Eclipse 的 Jar 文件，解压到合适位置，再创建快捷方式到桌面即可。

2. 以 《Spring 5 in Action》 中 Taco-cloud 项目为例，将其纳入 git 管理，参照 github官方 [maven ignore模板](https://github.com/github/gitignore/blob/master/Maven.gitignore)修改，比如使用无需 maven-wrapper.jar 的脚本。

3. 在常用浏览器安装 livereload 插件，这样调试 view template 时能和 Spring Devtools 启动的 LiveReload 服务器配合工作，全自动地完成网页刷新。

4. lombok 是写 bean 利器，可以直接通过 pom.xml 搜索 lombok 添加，maven 完成自动下载后，找到处于硬盘的位置，`java -jar lombok.jar` 运行（根据Eclipse安装位置不同，可能需要系统管理员权限），让IDE环境对简化写法脱敏，重启 IDE 刷新项目即可使用简化版领域对象。

5. Web template 的辅助输入插件不知道应该用什么？这部分经常有变量替换，在html中插入各种符号，`# @ $`，还是挺容易出错的。

其他常用快捷键：

`Alt + /` 内容辅助输入
`Ctrl + Alt + r` 查找打开文件

{{< highlight txt >}}
{{< /highlight >}}

封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>