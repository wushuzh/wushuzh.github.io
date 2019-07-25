+++
date = "2019-05-27T23:40:13+08:00"
title = "ARTS 2019w22"
showonlyimage = false
image = "/img/blog/arts-49-per-week/ml.webp"
draft = true
weight = 1922
tags = ["ARTS"]
+++

GOJEK 的 Kubeflow 落地实践
<!--more-->

## Algorithm

leetcode 上最简单的题目来了 771_jewels_and_stones


## Review


## Tip



## Share

看了 KubeCon 2019 来自 GOJEK 和 Google 两位工程师的演讲：

(Moving People and Products with Machine Learning on Kubeflow - Jeremy Lewi, Google & Willem Pienaar)[https://youtu.be/-GYiatVNemY]

讲述了 GOJEK 是如何在 k8s 上利用 kubeflow 部署他们的机器学习模型的。

- Traffic Manager 为不同模型部署分流请求；
- Orachestrator 综合单个模型的几个变种形式结果后，再决策返回；
- Kubeflow Pipeline 能很好的将编译打包、模型训练、验证等常规步骤自动化，方便后续部署；
- 不同区域有不同的法律要求，最好部署不同不同的集群；

<img alt="GoJek using Kubeflow" src="/img/blog/arts-49-per-week/GoJek-kubeflow.jpg" class="img-responsive">

封面图片来自 [Machine Learning](https://dribbble.com/shots/5467229-Machine-Learning) <a href="https://dribbble.com/bynd"><i class="fa fa-dribbble" aria-hidden="true"></i> Beyond</a>