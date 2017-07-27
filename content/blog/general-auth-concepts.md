+++
date = "2017-06-08T21:49:02+08:00"
title = "认证协议简介"
showonlyimage = false
image = "/img/blog/general-auth-concepts/id.png"
topImage = "/img/blog/general-auth-concepts/id.gif"
draft = false
weight = 92
tags = [ "telecom", "AAA" ]
+++

Diameter从属的大类是，认证协议的一种。
<!--more-->

介绍Diameter前，先退一步看一下大背景。

## 认证协议简介

通常意义上的认证协议

- 用于两点间传输那些和认证相关的请求、确认等消息
- 进一步包含对请求方自己声称的身份的确认（或验证）的过程

最简单的形式当然就是基于密码的认证：假设Alice和Bob都同意使用特定协议做身份认证，之前Bob的数据库中已保存了Alice的密码（或明文或其他形式）。

- Alice根据协议规则向Bob发送含有密码的报文
- Bob利用数据库中保存的密码对其做相应检查，然后发送认证成功或失败的结果报文

<img alt="alice and bob" src="/img/blog/general-auth-concepts/alice_bob.png" class="img-responsive" style="width:80%; height:80%; display:block; margin: auto;">

上述过程可能遭遇的安全威胁有：监听、回放攻击、中间人攻击、字典攻击、暴力破解等。为了应对这些攻击，大多数应用的协议都设计的更为复杂。

## 认证协议类型

比如开始是 PPP 也就是点对点的认证协议，认证过程常基于提前预设的密码，这类认证一般都是服务器客户端架构，同时随着黑客技术的提升，安全上的考虑也越来越完善：

  - 密码认证的协议 PAP ：最古老，但因为密文明文在网络中发送，易受监听和中间人攻击
  - 存疑握手认证协议 CHAP ：由服务器端发起，对发起时机和次数都不做限制。服务器先发送一串随机字符，客户端用这串字符和私有密码做为MD5函数的参数，将散列后的结果连同明文用户名一起发送。服务器端做相同计算并比较结果。
  - 可扩展的认证协议 EAP ：只定义一个框架，具体鉴权可以在其扩展中做各式定义。广泛使用在宽带，光纤网洛，局域或广域的无线网（IEEE 802.3 / 802.11/ 802.16)

<img alt="eavesdropping" src="/img/blog/general-auth-concepts/eavesdropping.jpg" class="img-responsive"
style="width:70%; height:70%; display:block; margin: auto;">

随着应用场景和伴生需求越来越丰富，如权限细分和计量计费，从A到AAA ( 你是谁，你可以干啥，你用了多久 )，认证协议也变得越来越复杂。典型场景当然就是移动通信网络，要应对的场景不仅仅是到跨省漫游，甚至出国漫游时，都要满足认证、鉴权、计量三方面的功能。

  - TACACS: 历史最为久远的AAA协议
  - RADIUS: 广泛被ISP互联网服务提供商使用，依然基于用户名密码，UDP传输
  - DIAMETER: 从RADIUS演进而来，使用TCP、SCTP可靠传输和TLS加密，是一个点对点的认证架构（非传统的客户端服务器的架构）

其他更为异类的，如MIT发明的Kerberos，比较复杂，需要单独讲述

## 单点登录
当需要的服务由不同网站提供时，用户可能需要做多次注册（因为你不了解网站的密码存储策略，全部使用相同的密码又是很不安全的做法），于是SSO就来解决这一问题。它实际起到了一个中间桥梁的作用，对不同的、彼此独立应用的认证机制做相应的适配。

相应的，你也要考虑清楚，因为有了SSO，你只需一个密码，这个密码就变得非常重要了。（英文叫法 keys to the castle 或 keys to the kingdom)，为了应对这种问题，业界建议是把SSO与OTP或智慧卡混合起来，同时服用。

进而衍生出的SAML，以及OAuth, OpenID再单独开篇讨论……


缩略语解释

> - AAA：Authentication, Authorization，Accounting 分别是认证、授权和计量
> - PPP: Point-to-Point Protocal
> - PAP: Password Authentication Protocal
> - CHAP: Challenge-handshake authentication protocal
> - EAP: Extenible Authorization Protocal
> - SSO: 单点登录
> - OTP: One-time password 一次性密码

参考文档

> - https://en.wikipedia.org/wiki/Authentication_protocol
> - https://en.wikipedia.org/wiki/Single_sign-on
> - https://en.wikipedia.org/wiki/Authentication
> - https://youtu.be/Rt3aMJL6e0g
> - https://en.wikipedia.org/wiki/One-time_password

封面图片来自 [HUD](https://dribbble.com/shots/3646248-HUD) <a href="https://dribbble.com/bifftenon"><i class="fa fa-dribbble" aria-hidden="true"></i> Biff Tenon</a>  

文中图片来源网络
