+++
date = "2017-12-07T13:17:31+08:00"
title = "网页式的演讲文稿"
showonlyimage = false
image = "/img/blog/make-slides-by-markdown/presentation.png"
draft = false
weight = 601
+++

转化 Markdown 为 HTML slides
<!--more-->

### 安装

{{< highlight console >}}
mkdir ~/Document/slides 
cd ~/Document/slides
yarn add remarker
{{< /highlight >}}

### 文档结构

#### 页面分隔

- 标准页分割符: ---
- 增量页分割符: -- 新页内容 = 前一页内容 + 新内容

{{< highlight markdown >}}

{{< /highlight >}}

#### 页面属性

每个页面的初始行若为 key: value 形式则被认为是此页属性，特定键包括:

- name 键值用于命名当前页面，使用 markdown link 语法即可做页面跳转 [Go2Aslide](#slidenamevalue)
- class 键值接收多个逗号分割的值，比如可选用 left, center, right, top, middle, bottom 来对齐内容
- background-image 用于设定背景图
- count 值为 false 表示当前页不计入总页数

{{< highlight markdown >}}

{{< /highlight >}}

参考文档

> - Kat Marchán (2017-07-11) [Introducing npx: an npm package runner](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)

封面图片来自 [Presentation](https://dribbble.com/shots/3007989-Presentation) <a href="https://dribbble.com/Frizler"><i class="fa fa-dribbble" aria-hidden="true"></i> Anton Fritsler (kit8)</a>
