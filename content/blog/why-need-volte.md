+++
date = "2017-06-01T20:09:06+08:00"
title = "为什么需要VoLTE"
showonlyimage = true
image = "/img/blog/why-need-volte/xkcd_1422.png"
draft = false
weight = 0

+++

![volte-word-cloud](/img/blog/why-need-volte/VoLTE_Charging_Guide-Thumbnail.jpg)

进入移动时代，我们先是通过 2G 实现了可以“移动着”接打电话；之后的 3G 使得“移动着”上网也变得可能；现在借助 4G 网络，我们甚至可以用手机去参与各大电商购物节的秒杀活动了。作为一个全 IP 包交换的网络，LTE 设计的全部焦点放在如何用最小的延迟提供最大的数据吞吐率，主要在以下两个方面发力：

> 1. 如何与公共数据网的连接
> 2. 如何应对终端的移动性

而对于传统语音服务（接打电话），它直接忽略了。

![2G-vs-3G-vs-4G](/img/blog/why-need-volte/Compare-1G-2G-3G-4G-5G.jpg)

假设你有一台普通智能手机，并且还未与运营商注册使用VoLTE业务，你可以在接打电话的时候注意观察手机屏幕，在整个通话过程，无论拨号还是振铃，一直到你挂断前，平时通常位于屏幕右上角的4G标志会暂时消失不见。你被实际是暂时回退( CSFB ) 到 2G 或 3G 网络完成这些电话功能。所以如果你想边打电话边同时上网查查对方说的约会地点在哪里，做个路径规划或是查看下沿途路况，你会看到地图 App 告诉你“网络不畅，请稍后重试”。

<img style="width:30%; height:30%; display:block; margin: auto 10%;" src="/img/blog/why-need-volte/404_big.png">

到了 2010 年，两大国际组织 GSMA 和 3GPP 共同定义了 VoLTE 的通信标准，目标是使得我们不但可以在 4G 网络之上实现语音短信服务，而且还外赠视频通话( ViLTE ) ，多方电话会议等其他服务。拿最常见的电话服务来说，普通用户能感知到两点，一、新的编码技术使得话音质量提升 40 % ；二、呼叫接通时延减小 50 % (从按下拨号到对方振铃)；

实现了上述标准的系统叫 IP多媒体子系统( IMS )。由于前面提到 4G 网络已经是一个全 IP 通信网，因此 IMS 也就很自然的利用很多在互联网领域应用地较成熟的 IETF 各种协议：SIP、MRSP、Diameter、IPSec、RTP、RTCP……

![LTE-IMS](/img/blog/why-need-volte/VoLTE-end-to-end-architecture.jpg)

后续我们会介绍一下为了解决上述问题引入的 IMS 系统，并重点介绍一下用信令传输的 SIP 协议。

缩略语解释

> - LTE：和4G等同，GSM、CDMA、UMTS 代指第二、三代移动通信
> - CSFB: CS Fall Back 对于如何在4G网络上做传统语音传输并不清晰，厂家各自为政，3GPP选择将回退到电路域提供语音服务作为一个中间过渡方案
> - GSMA：GSM协会的成员包括全世界运营商（甲方）和其周边厂家，当前其主攻的三方向是“
未来网络”，“物联网”，“电子身份”
> - 3GPP: 第三代合作伙伴计划，最初是为了3G技术演进方向与另一个由高通主导的3GPP2分庭抗礼，多数电信设备制造商（乙方）都参与其中
> - ViLTE: Video over LTE
> - IETF: 互联网工程任务组，这是一个开放的标准化组织，非盈利性质使得它看上少了些铜臭气

参考文档

> - http://www.3glteinfo.com/volte-voice-lte/
> - http://www.3glteinfo.com/volte-history-timeline/
> - https://realtimecommunication.wordpress.com/2016/06/22/volte-illustrated-beginners-guide/

文中图片来源网络
