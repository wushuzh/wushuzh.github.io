+++
date = "2017-11-28T20:44:11+08:00"
title = "Nmap 扫描"
showonlyimage = false
image = "/img/blog/use-nmap-to-scan/site_zero.png"
draft = false
weight = 110
+++

成片扫描网段的快捷方法
<!--more-->

### 代码覆盖

{{< highlight console >}}
nmap -T5 -sP 135.252.165.2-255
nmap -sS -p 22 135.252.165.2-255
{{< /highlight >}}


参考文档

> - []()

封面图片来自 [Site Zero ///](https://dribbble.com/shots/3751115-Site-Zero) <a href="https://dribbble.com/mjmurdock"><i class="fa fa-dribbble" aria-hidden="true"></i> mike murdock</a>
