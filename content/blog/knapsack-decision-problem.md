+++
date = "2017-12-06T20:11:31+08:00"
title = "决策树:背包问题"
showonlyimage = false
image = "/img/blog/knapsack-decision-problem/gear.png"
draft = false
weight = 402
+++

如何高效解决经典的背包问题
<!--more-->

### 背包问题

- 已知一系列物品的价值和占用空间
- 只有一个大小有限可以存放上述物品的背包
- 如何在这个有限大小的背包中放入尽量多价值的物品

### 决策树

- 从第一件物品开始，加入背包对应根节点的左侧分支，放弃对应右侧
- 第 n 件物品对应树的第 n 层
- 每条路徑对应一种方案: 从路徑可还原当下方案加入/放弃物品的数量

> n + 1 层会是第 n 层的幂集 (Power set)

参考文档

> -

封面图片来自 [Digital Nomad's Gear](https://dribbble.com/shots/2669062-Digital-Nomad-s-Gear) <a href="https://dribbble.com/DanDragomir"><i class="fa fa-dribbble" aria-hidden="true"></i> Dan Dragomir</a>
