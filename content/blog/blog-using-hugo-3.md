+++
date = "2017-07-10T19:08:01+08:00"
title = "像雨果一样写作 3"
showonlyimage = false
image = "/img/blog/blog-using-hugo-3/cartography.png"
draft = false
weight = 22
tags = [ "Blogging", "Hugo", "Image" ]
categories = [ "Writing" ]
series = [ "Blog like a Pro" ]
+++

想写论文？那看看如何插入代码、图及公式。
<!--more-->

## 插入代码

实现代码语法高亮有两个选择：  

1. 提前在本地安装 Pygments ，并将程序 pygmentize 目录加入 PATH，这样 Hugo 就会在生成网页或 RSS 的时候，将被 shortcode highlight 封装起来的代码渲染好。  
2. 在生成页面中加入语法高亮的 js 库，将渲染工作推到客户端去做，这样的好处是可能支持更多语言  

在 Markdown 中写成  
```
{{</* highlight go */>}}
package main
import "fmt"
func main() {
    fmt.Println("hello world")
}
{{</* / highlight */>}}
```
最终得到语法高亮的效果
{{< highlight go >}}
package main
import "fmt"
func main() {
    fmt.Println("hello world")
}
{{< /highlight >}}

## 插入流程图

[knsv/mermaid](https://github.com/knsv/mermaid) 是一个在 GitHub 上被加星了 13k+ 的项目：它提供了类 Markdown 的方式，先通过文本形式，用特定语法描述图中元素和关系，再将其渲染为对应各类图。

{{< figure src="/img/blog/blog-using-hugo-3/flow_chart.png" title="Flow Chart" >}}

{{< figure src="/img/blog/blog-using-hugo-3/seq_diagram.png" title="Seq Diagram" >}}

{{< figure src="/img/blog/blog-using-hugo-3/gantt_diagram.png" title="Gantt  Diagram" >}}

{{< figure src="/img/blog/blog-using-hugo-3/class-diagram.png" title="Class   Diagram" >}}

{{< figure src="/img/blog/blog-using-hugo-3/gitgraph.mm.png" title="Git Graph" >}}

## 插入公式

TODO

## 插入表格

TODO

Name    | Age
--------|------
Bob     | 27
Alice   | 23

<div class="table-responsive">
  <table class="table">
     <thead>
        <tr>
           <th>Name</th>
           <th>Age</th>
        </tr>
     </thead>
     <tbody>
        <tr>
           <td>Bob</td>
           <td>27</td>
        </tr>
        <tr>
           <td>Alice</td>
           <td>23</td>
        </tr>
     </tbody>
  </table>
</div>

缩略语解释

参考文档
