+++
date = "2020-03-09T17:20:39+08:00"
title = "ARTS 2020w10"
image = "/img/blog/arts-91-per-week/"
draft = true
weight = 2010
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip



## Share

Auto scaling

    ELB

    public-subnet-1   public-subnet-2

    cloudwatch


    demo

    1. what to launch (launch configuration: AMI, user-data, sg)
    2. where to launch
        auto scaling target group: 1. above  
        group size : 2 (per az)
        assign VPC and two pub subnet

    3. when to launch: scaling policy
        min/max instance number
        threshold

Security (Share responsibility model)



User Data   
============
Application   RDS，aws 保证
GuestOS   aws提供用户多种加密、安全的选项
------------
Hypervisor
Network
Physical

Cost

AWS Pricing Calcator
AWS cost explorer 各种报表
AWS budget 告警
AWS Trust advisor 各种优化建议：备份


封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>