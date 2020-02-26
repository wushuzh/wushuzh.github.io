+++
date = "2020-01-06T08:45:46+08:00"
title = "ARTS 2020w01"
image = "/img/blog/arts-82-per-week/"
draft = true
weight = 2001
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

用 7zip File Manager (GUI) 解压含中文的文件夹或文件时，出现乱码。但通过命令行 7z 添加 mcp 参数可以解决这个问题。详见 [#2198 Unable to extract MBCS encoded filenames](https://sourceforge.net/p/sevenzip/bugs/2198/)

`7z x a.zip -mcp=936`

究其根源，这个问题一个 Windows 10 的 beta 特性的开关有关，这个开关在控制面板的区域与语言设置中，一般对当前系统区域设置都为：“中文（简体、中国）”，这是大家都一样的设置，但是下面还有个可选的bata特性，叫做“Beta版：适用 Unicode UTF-8 提供全球语言支持”，不知我是什么时候将其勾选生效，结果导致各种软件中文解析乱码。

详见 CSDN leehaming 2018-09-11 [修改windows的默认编码](https://blog.csdn.net/lee_ham/article/details/82634411)

`chcp`
`chcp 936 # 7zip excel vba打开正常`
`chcp 65001 # 上述软件遇到中文乱码或崩溃`

## Share



封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>