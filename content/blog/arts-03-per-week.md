+++
date = "2018-07-16T21:31:01+08:00"
title = "ARTS 2018w29"
showonlyimage = false
image = "/img/blog/arts-03-per-week/game.png"
topImage = "/img/blog/arts-03-per-week/game.gif"
draft = false
weight = 1829
+++

链表入门、以及工程师的能级跃迁
<!--more-->

## Algorithm

本周解析的题目是 [83. Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/)

题目通过注释给出了数据结构的最简实现，增加的 `dunder str` 方法纯粹是为了打印调试。

{{< highlight python >}}
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

    def __str__(self):
        return 'ListNode(' + str(self.val) + ')'


def printSSL(lst):
    if lst:
        print("{} -> ".format(lst), end="")
        printSSL(lst.next)
    else:
        print('None')

{{< /highlight >}}

首先是形成测试数据集，python list 是直觉之选，但使用时要注意一些操作的特殊特性，比如

{{< highlight py >}}
>>> s = [[42]]*3
>>> s
[[42], [42], [42]]
>>> s[0].append(0)
>>> s
[[42, 0], [42, 0], [42, 0]]
{{< /highlight >}}

向链表的添加元素操作，如果是向头部添加比向尾部添加容易的多，所以可以先将 list 倒序在逐一添加，最终获得头部元素 head;

{{< highlight py >}}
def cons(v, tail):
    head = ListNode(v)
    head.next = tail
    return head


def mklist(*args):
    return reduce(lambda tail, v: cons(v, tail), reversed(args), None)

{{< /highlight >}}

将链表转换回 list 的方法
{{< highlight py >}}
def convert_array(head):
    a = []
    succ = head
    while succ:
        a.append(succ.val)
        succ = succ.next
    return a
{{< /highlight >}}

此时就可以模拟生成任意个排好序的链表作为测试数据集了

{{< highlight py >}}
def test_multi(check_itr=1000):
    for _ in range(check_itr):
        a = []
        no_dup = randint(2, 100)
        for i in range(no_dup):
            repeat = randint(1, 3)
            a += [i] * repeat
        new_head = deleteDuplicates(mklist(*a))
        assert convert_array(new_head) == list(range(no_dup))
{{< /highlight >}}

本题解法：

{{< highlight py >}}
 def deleteDuplicates(head: ListNode) -> ListNode:
    cur = head
    while cur and cur.next:
        if cur.next.val == cur.val:
            cur.next = cur.next.next
        else:
            cur = cur.next
    return head
{{< /highlight >}}


另外看了一下 leetcode 的专题学习卡片，也许这是开始做相关的数据结构试题前，入门的好方法，比如[链表](https://leetcode.com/explore/learn/card/linked-list/)。

## Review 

本周刷微博，看到 John Carmack 这个名字，然而我并不知道是这是哪位大神——除了 Linus Torvalds 以外，CS名人堂的前辈我居然说不出半打，简直就是个外行。刺激之下就在 wiki 上读了一下他的[词条](https://en.wikipedia.org/wiki/John_D._Carmack)，因为我对计算机图形学和游戏开发比较陌生，就摘录一些有趣的部分。

John D. Carmack 于 1970 年出生，父亲是本地电视台新闻记者，8、9 岁时通过游戏厅接触到了那些最早的射击、迷宫类的经典游戏，对宫本茂很是仰慕。14 岁被劳教一年，原因是帮一群孩子偷 Apple II 电脑，在“行动”中他成功地使用铝热法融化了学校的窗户，却因为一个猪队友的开窗操作触发警报而导致被捕，他大学仅上了 2 个学期，然后就退学去做了程序员。在 Softdisk 公司工作时，开发了游戏[“指挥官基恩”](https://en.wikipedia.org/wiki/Commander_Keen)，也认识了 [John Romero](https://en.wikipedia.org/wiki/John_Romero), [Adrian Carmack](https://en.wikipedia.org/wiki/Adrian_Carmack) 等人；21 岁时创建 [id Software](https://en.wikipedia.org/wiki/Id_Software) 公司，之后领导开发了 [“重返德军总部 3D”](https://en.wikipedia.org/wiki/Wolfenstein_3D)，[“毁灭战士”](https://en.wikipedia.org/wiki/Doom_(1993_video_game))，[“雷神之锤”](https://en.wikipedia.org/wiki/Quake_(video_game))，[“狂怒”](https://en.wikipedia.org/wiki/Rage_(video_game))等经典游戏。

由 Carmack 率先使用或推广的技术有

- [adaptive tile refresh](https://en.wikipedia.org/wiki/Adaptive_tile_refresh)
- [raycasting](https://en.wikipedia.org/wiki/Raycasting)
- [binary space partitioning](https://en.wikipedia.org/wiki/Binary_space_partitioning)
- [suface caching](https://en.wikipedia.org/wiki/Surface_caching)
- [Carmack's Reverse](https://en.wikipedia.org/wiki/Shadow_volume)
- [MegaTexture](https://en.wikipedia.org/wiki/MegaTexture)

他开发的图形引擎还被授权使用在[“半条命”](https://en.wikipedia.org/wiki/Half-Life_(video_game))，[“使命召唤”](https://en.wikipedia.org/wiki/Call_of_Duty)，[“荣誉勋章”](https://en.wikipedia.org/wiki/Medal_of_Honor_(1999_video_game))等游戏上。

2000 年，当他发现将他在定制法拉利上花的钱，放在火箭研究上足以完成很多事情后，就成立了 [Armadillo Aerospace](https://en.wikipedia.org/wiki/Armadillo_Aerospace) 公司，每年从自己腰包中拿出一百多万美元支撑公司研发运营，然后在 2008 年参加了 NASA 举行的[月球垂直起落飞行器挑战赛](https://en.wikipedia.org/wiki/Lunar_Lander_Challenge)，获得常规难度组的第一名，并在次年完成了该比赛的终极难度挑战，获得相应奖金。 

2013 年底 Carmack 正式离开 id ，作为 CTO 加入 Oculus VR 公司。这之后 id 的母公司 ZeniMax 将 Oculus 和其收购者 Facebook 告上法庭，认为其侵犯了前者的知识产权，最终陪审团认为除了 Carmack 之外的其他相关人等都有责任，需要对 ZeniMax 支付赔偿。

Carmack 一直推崇开源软件的理念并付诸实践:

- 1995 公布“重返德军总部 3D”源代码；
- 1997 公布“毁灭公爵”源代码；
- 1996 一位和 id software 无关的程序员基于“雷神之锤”被泄露的源码将游戏迁移到 linux，还将补丁发给 Carmack ，Carmack 则指示基于这个补丁来实现官方迁移，而没有对补丁作者采取任何法律行动；

Carmack 同样会称赞其他的游戏图形引擎，比如 [Ken Silverman](https://en.wikipedia.org/wiki/Ken_Silverman) 的 [Build Engine](https://en.wikipedia.org/wiki/Build_(game_engine))、[Tim Sweeney](https://en.wikipedia.org/wiki/Tim_Sweeney_(game_developer)) 的 [Unreal Engine](https://en.wikipedia.org/wiki/Unreal_Engine) 。

还有就是 Carmack 很喜欢披萨，他在 id 工作期间，几乎每天都会从达美乐叫一份中号意大利辣香肠披萨，达美乐 15 年来从未对 Carmack 的订单涨过价，披萨也一直是由同一名的快递员送达。

如果对 John Carmack 和 id software 由更多兴趣，建议阅读 2012 年 slashdot 上的[专访](https://games.slashdot.org/story/99/10/15/1012230/John-Carmack-Answers)以及 David Kushner 的书[《 Masters of Doom 》](https://en.wikipedia.org/wiki/Masters_of_Doom)

## Tip

本周继续学习 Kafka ，全局图景仍未梳理出来，但看和安全相关的 TLS/SSL 部分真是十分复杂，先将数据加密的步骤列出来，后续实践后再补充原理和细节。

**加密原因**：未经加密的数据对途径的路由完全透明，所以一般发送方要先对数据进行加密再发送，而接收方收到数据后，会先解密再使用。要想实现 Java 服务器客户端的 TLS/SSL 数据加密传输，还要借助一个可以被广大人民群众信任的 CA ，它另一个职责是负责为服务器端颁发签名证书。

我们可以通过下面指令模拟一个私立 CA ,产生文件为 ca-cert 和 ca-key

{{< highlight shell >}}
openssl req -new -newkey rsa:4096 -days 365 -x509 -subj "/CN=My-Security-CA" -keyout ca-key -out ca-cert -nodes
{{< /highlight >}}

之后轮到希望实现数据加密的服务方，由它自己先创建一个 keystore (myserver.keystore.jks) 并借此为自己生成一个证书 cert-file ，然后将这个文件(和密码？)通过邮件发送给 CA
{{< highlight shell >}}
export SRVPASS=sercet

keytool -genkey -keystore myserver.keystore.jks -validity 365 -storepass $SRVPASS -keypass $SRVPASS -dname "CN=myserver-dns" -storetype pkcsl2

# keytool -list -v -keystore myserver.jks

keytool -keystore myserver.keystore.jks -certreq -file cert-file -storepass $SRVPASS -keypass $SRVPASS
{{< /highlight >}}

CA 借助自己的证书和密钥，对收到的证书进行签名，产生 cert-signed 文件，发回给服务器申请方

{{< highlight shell >}}
openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:$SRVPASS

# keytool -printcert -v -file cert-signed

{{< /highlight >}}

服务器端将收到的证书导入 keystore 中，并向 JAVA 应用告知 keystore 和 truststore 的位置和访问密码。

{{< highlight shell >}}
# keytool -keystore mysever.truststore.jks -alias CARoot -import -file ca-cert -storepass $SRVPASS -keypass $SRVPASS -noprompt
# keytool -keystore mysever.keystore.jks -alias CARoot -import -file ca-cert -storepass $SRVPASS -keypass $SRVPASS -noprompt
keytool -keystore mysever.keystore.jks -import -file cert-signed -storepass $SRVPASS -keypass $SRVPASS -noprompt
{{< /highlight >}}

此时在任意一个客户端试图访问相应端口测试，只要它的 truststore 有 CA 的证书，并告知 JAVA 客户端 trustore 文件位置和访问密码即可实现加密传输数据

{{< highlight shell >}}
openssl s_client -connect myserver-dns:sslport

# export CLIPASS=clipass

# keytool -keystore myclient.truststore.jks -alias CARoot -import -file ca-cert -storepass $CLIPASS -keypass $CLIPASS -noprompt

keytool -list -v -keystore myclient.truststore.jks
{{< /highlight >}}


## Share

本周阅读维基百科 John Carmack 词条的时候，回忆起小时候我也很喜欢玩游戏，但最复杂的操作也只限于使用内存修改工具修改金钱，或是在 config.sys 中添加诸如 memmaker 的调用，使得我的 486 DOS 系统能跑起来一个新的需要更多内存的盗版游戏。大家中学时肯定都学过铝镁反应，但 Carmack 居然用来融窗户……人和人真的很不一样。

以前我无视房地产行业，对任志强的了解仅限于新闻转载了他的“开炮”言论——然后听说是说的太过火而受到党内处分。这周课余时间读了几页任志强的《野心优雅》，才知道这位任大炮年轻时也算是“白手起家”，然后也蹲过看守所，后来又用十几年的时间，将一个注册资金 1500 万的公司做成了净资产 30 亿，总资产 80 亿的明星地产企业，再后来二次创业……这是一位能在退休的时候“惊动”国家总理做出批示的老总。看他复盘自己日常决策和思考，才觉得他身处环境和所面对问题的复杂程度不是我这种职场小白能想象的，无论是面对强势的资本、保守的政府、还是企图将其踩下的后辈，都能坚持原则，进退有度，令人钦佩。

吴军老师在《5级工程师和职业发展》中提到前苏联物理学家[朗道](https://en.wikipedia.org/wiki/Lev_Landau)有一个小名单，用于将他所知的物理学家进行分级 1 - 5 级，特殊的有牛顿，在最高 0 级，爱因斯坦 0.5 级……吴军老师仿照此法也将工程师划分为 5 级：

- 第五级： 能独立解决问题，完成工程工作；
- 第四级： 能指导和带领其他人一同完成更有影响力的工作；
- 第三级： 能独立设计和实现产品，并且在市场上获得成功；
- 第二季： 能设计和实现别人不能做出的产品，也就是说他的作用很难取代；
- 第一级：开创一个产业；

虽然表面上大家都互相张工、王工、李工的叫，但事实上相邻级别之间的输出成果最小估计也能差出十倍(指数级)。读完吴军老师的文章才体会到工程师这个头衔的重量，按吴军老师的说法，“4 级工程师在中国其实非常缺乏，不少人之所以得到机会，是因为他们和上下级之间较强的沟通能力帮助了他们”。


封面图片来自 [Gears Ink](https://dribbble.com/shots/2967240-Gears-Ink) <a href="https://dribbble.com/jayemsee"><i class="fa fa-dribbble" aria-hidden="true"></i> joshua corliss</a>