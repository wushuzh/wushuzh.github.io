+++
date = "2017-07-27T18:31:59+08:00"
title = "番外篇——密码演化史"
showonlyimage = false
image = "/img/blog/pitfalls-of-the-passwd/try-passwd.png"
topImage = "/img/blog/pitfalls-of-the-passwd/try-passwd.gif"
draft = false
weight = 93
+++

这些年我们用过的那些密码……
<!--more-->

作为 PSP 安全审计，了解和学习安全方面的知识

原文地址 https://www.troyhunt.com/passwords-evolved-authentication-guidance-for-the-modern-era/

## 密码误区

遵循官方指导或科技巨头 NIST & NCSC 和 Microsoft

密码不仅仅做能否登录系统这样的二元论判断。
现代的区分自动攻击、暴力破解
存疑登录后的权限再验证

1. 别限定长度，别截断
    限定长度会让密码管理器无法使用，并且影响可用性，见实例
      https://www.troyhunt.com/security-insanity-how-we-keep-failing-at-the-basics/
    客户会质疑的是否对于密码没有做足够的存储规划：加密过的密码都变为相同长度，限定长度只会让人觉得你是在用明文和比较的固定长度的表字段保存密码。无论64个字符，256个字符都可以，因为散列后字符都会变为一样的长度。
2. 别用复杂规则折磨用户
    某些特殊字符和一些攻击相关，比如 <> 被用于 XSS 攻击；单引号别用于 SQL 注入攻击；但密码是绝不能在 UI 中被显示出来的，另外密码被散列化之前绝不能提交到后端数据库，散列化后就只剩字母和数字了。因此可打印的 ASCII 和空格都应该可以被接受，Unicode 也应该被接受。另外将规则显示出来也很不可取。黑客经常会利用这些规则。而客户不断尝试后弄了个勉强符合规则的弱密码，还对你的网站很生气。这个密码可能会被用到多处。
3. 还在要用户做密码提示设置？
4. 拥抱密码管理器
5. 方便用户粘贴密码
6. 别强制用户定期修改密码
7. 向用户报告异常登录事件
8. 将已泄露、不安全的密码进行屏蔽
9.



## HTTPS 和 DPI 的没落


封面图片来自 [Fossils trouble](https://dribbble.com/shots/2448430-Fossils-trouble) <a href="https://dribbble.com/cjiabka"><i class="fa fa-dribbble" aria-hidden="true"></i> Yaroslav Kuryanovich</a>  

文中图片来源网络
