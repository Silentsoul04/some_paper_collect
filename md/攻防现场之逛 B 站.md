> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4DyZPdXrlXqGez3OXkLEEA)

男子攻防现场为何逛 B 站？

这一切背后是安全的泯灭，

还是靶标的沦丧？

还是封 IP 之后心灵的扭曲？

欢迎收看本期《红队观察》

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KF0MvJ6wYicAalCDGeaMgnmYEoibxEWibYZ4MJR9m6zfMVwrRT6xIVlayoA/640?wx_fmt=png)

    在某次 HVV 尾声，面对手头的目标实在啃不动

    于是对统一身份认证口令进行尝试

    靶标是一个 211 大学，为了不进局子，出现的图片打码会很严重

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KFL8g4uxlSgCWGx5wObqDlPXCPCEYGtQgYyd56fmYcrBriaK9YrvCV8ibA/640?wx_fmt=png)

    首先账号的格式比较常见：入学年份 + 专业编码 + 班级学号，初始密码是身份证后六位。

    进行一下爆破，发现存在限制 (错误五次账号将限制登陆) 随后进行了统一身份认证常规性的信息收集，主要是贴吧，QQ 群，和学校新闻等，因为在贴吧遇到过太多卧龙凤雏，所以我很喜欢在里面找账号密码。

    但是显然这个大学没啥人玩贴吧（一般职高等玩贴吧人多一些）

在 QQ 群内也没有收获，可能是没舍得花钱的原因。

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KFjsyNrcy6mZcmrOXHH88iak1zf17sVDSgLuibGq52gSELIyQ9MTD5y5Sg/640?wx_fmt=png)

    一筹莫展之际，开始思索一所 211 大学和一所普通大学的区别，并无任何歧视普通大学意思，单纯从心理分析。因为我认为弱口令和信息泄露类的漏洞产生很大原因是因为一些人性的弱点（ps：这么说还挺有逼格的）

我想了一些可能存在信息泄露的点：

    1、一所规模更大的大学新闻报道会更多，极有可能泄露一些姓名，入学年份和专业。

    2、学生的数量，算上研究生博士生和历届毕业生，拥有统一身份认证账号的人数其实是非常多的

    3、优越感，平心而论我要考到清华我绝对也得发个朋友圈装个逼，可能是我比较肤浅，但是肤浅的人肯定不止我一个。

    对于第一点，确实找到了很多报道，获得很多人名，专业，入学年份，通过对专业编号的搜索还有学号规律可以确定大概账号区间，但是身份证没法获得于是卡住。

    最终突破口还是在 “优越感”，在思索如何进一步拓展我脑海里浮现了一个词 “开箱”，因为经常刷 B 站，抖音等看过很多类似 “手机开箱 “或者一些新鲜东西的开箱，又结合某次某同学考上很牛逼的大学朋友圈晒录取通知书。

    想到尝试一下，于是出现了文章开头的一幕

    我在攻防现场点开了 B 站

    强忍着看两集番的冲动搜索了 “xx 大学开箱 “

    结果不出所料，有很多

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KFzFy4aX0TtWicZsFvSUOYaqkqRCqun954QRV6qWa47iciceHFiaatYyiceBA/640?wx_fmt=png)

    在挨个翻看的时候发现了某个视频在拍摄过程中打码在切换时有一秒消失

阅片多年，暂停技巧早已拉满。

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KF8HohEX6XjmzWT4ZYBprxn9jHkicrkJrHr6dRibrHygudILwumTSDNCow/640?wx_fmt=png)

    学号直接暴露，并且附带了身份证号码  

    旁边同事略带不可思议的目光看着 B 站，我冲他摆了摆手：“常规操作。“

    成功登录，拿着账号测试可以访问那些网站进行刷分。

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KFWrZkUsQpiadQZuahml9icFB8JuMMWDvAImbZPRoAM5aKWV5IROhnu7OQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MZsbTyQzgOnbRwujoLNR8KFEdFLicZsljwMVcJ4M2RVTVOz9Y7NnicwBIxTkRo22vpD0ePiaFjZicJCnA/640?wx_fmt=png)

    我所理解的信息收集并不仅仅是用工具跑一边子域名，目录，端口这些常规的，而是在信息收集的过程去分析这个网站所呈现的一些特性，就像前文提到过的人性的弱点，尤其是弱口令，“图方便” 这三个字产生了多少漏洞。

对我而言渗透的乐趣在于发掘别人未发现的点。

-END-

![](https://mmbiz.qpic.cn/mmbiz_gif/eqGGHicCG3MaY6g8c4royZZdoEQzaibpp7hWBdZDHukT8RoSoK1MU9vDWyA2mFUrwpHW1hR66bIH2dD1fKj1L0PA/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_jpg/eqGGHicCG3MaY6g8c4royZZdoEQzaibpp7dMwH9adLvutCcLicdEPPFyr9jnN06NLRu6Jg85SEicmhbc53z5wXZzWQ/640?wx_fmt=jpeg)

微信号：Zero-safety

- 扫码关注我们 -

带你领略不一样的世界

![](https://mmbiz.qpic.cn/mmbiz_gif/eqGGHicCG3MaY6g8c4royZZdoEQzaibpp7LVpnnataVgcuqkaiaxbuf1mcdBkYc7ElZZpK9QLslbZZxMMRehafjQA/640?wx_fmt=gif)