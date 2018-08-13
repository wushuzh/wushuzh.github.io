+++
date = "2017-07-10T19:08:01+08:00"
title = "像雨果一样写作 3"
showonlyimage = false
image = "/img/blog/blog-using-hugo-3/cartography.png"
draft = false
weight = 22
tags = ["Hugo"]
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
{{</* /highlight */>}}
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

Hugo 通过 LaTex 语法，借助一个 JavaScript 库 [MathJax](https://www.mathjax.org/) 在各个浏览器中无差别地显示数学公式。在 Lincoln. A (2016-08-07) [Better TeX math typesetting in Hugo] (http://www.latkin.org/blog/2016/08/07/better-tex-math-typesetting-in-hugo/) 博客中相关介绍很详细，一步步跟着做即可，这里不重复，大体步骤如下：

1. 在文章显示模板中引入 MathJax 库
2. 自定义一个 Shortcodes 命名为 tex
3. 在自己的博客正文中调用新定义的 tex mathematics

> 另外，如果使用 Lincoln 的 tex 定义，就算浏览器禁用 JavaScript ，相关公式也能显示正常(通过图片)

例如，在 Markdown 中写成
```
{{</* highlight LaTex */>}}
{{</* tex "\sum_{n=1}^{\infty} 2^{-n} = 1" */>}} # 行文行内公式
{{</* tex display="\int \frac{1}{x} dx = \ln |x|" */>}} # 独占一行公式
{{</* / highlight */>}}
```
最终显示效果如下：

1. 行文行内公式 {{< tex "\sum_{n=1}^{\infty} 2^{-n} = 1" >}}
2. 独占一行公式
    {{< tex display="\int \frac{1}{x} dx = \ln |x|" >}}

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

> - 有道云笔记 (2016-05-10) [有道云笔记PC4.9](http://note.youdao.com/iyoudao/?p=1895)
> - 有道云笔记 (2016-04-07) [进击的Markdown教程]( https://mp.weixin.qq.com/s?__biz=MjM5NjAyNjkwMA==&mid=2723941495&idx=2&sn=eb542a9e934e3c4bbb9be4051037b73b&scene=4#wechat_redirect)
