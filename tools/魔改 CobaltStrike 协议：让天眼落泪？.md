> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NcqAROsqG3XsaLz4qO_LWw)

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawicF2cr8LRBiaia2ibQJ3LFpMGBQl6l9acTIKgU65qUIm3iaCuaticGWvLg5w/640?wx_fmt=gif)

**魔改 CobaltStrike 协议：让天眼落泪？**

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawL3vTPm2NWFdWwLhCoehBzQZicETuxSbLWlRia7KtcOjrz8Lqt6ZvXYTg/640?wx_fmt=gif)

  

---

1

概述

这次文章将较详细介绍下 Cobalt Strike 4.1 的 agressor 端登录 teamserver 端的通讯和 beacon 端与 teamserver 端通讯，包括元数据、心跳包、执行任务等数据传输。帖子写到一半 IDEA 授权就掉了，所以就拖到现在才写完，附项目地址：https://github.com/mai1zhi2/CobaltstrikeSource/

2

aggressor 端登录 teamserver 端的通讯分析

**2.1****、****agressor** **客户端发送密码登录** **teamserver** **端进行验证：**

运行 Teamserver 端，首先会创建 SecureServerSocket 对象，然后循环调用 acceptAndAuthenticate() 接受认证信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawIgugM2LZjViaUCibhhJuAfgNcqSW5wCKJT9tQiaTfjEzhQQh6fY9xtcBQ/640?wx_fmt=png)

点击 connet 按钮后：

Aggressor 端首先在 dialogAction() 方法获得相应的登录信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw0gibrv3ia5vzd9rwVkFdx7rluaJYV7ld7kn4ibj24bIOic2BKibJIyRIpwg/640?wx_fmt=png)

然后传入 host、port 创建安全套接字：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaws9C6UEbTXzyBh6ibYrK5613VhEfhvgjLXItRXQoKiaYxlxs127fxkdqg/640?wx_fmt=png)

SecureSocket 的构造方法中设置了相关的 socket 属性, 调用 createSocket() 传入要连接的 ip 地址和 port:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw4hEUEnYoiaknE89ANextyp5U1REt4nhlG2R50EZBQQxlicfunUkhblRg/640?wx_fmt=png)

agressor 端调用完 createSocket() 后，teamserver 端 acceptAndAuthenticate() 会接受到信息，其中跟入 authenticate(Socket var1, String var2, String var3) 方法负责处理接受到的信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawPRtyntfhHl73GjJ2hdhxU1mjYRPYx6KNbuZNvias9xPdZ9s9d5pxNUQ/640?wx_fmt=png)

跟入该方法，当 var4 没读取到足够的字符就一直阻塞：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawrAAicH2wCccib1VKVjAPtMXaxLllfbLo2icUsFsofu4ttf5nsBpDyFdbA/640?wx_fmt=png)

Aggressor 端构建完 SecureSocket 对象后，调用其 authenticate(string var1) 方法，并传入登录的密码，该方法主要通过密码构造相应的数据包，其数据包先传入 48879 这个数值（标志），然后再传密码的长度，接着传入密码的内容，最后以 0x41 进行填充：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawiawvzhkyGTymWqnTRFSK4ic5MBYkO6pFvPTlbrA8CA9yMXb0hhl2h2DQ/640?wx_fmt=png)

数据包 var3 内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawtZMuSjNR5qr1XzzX7bhCAk2x2Xxb9dK2ywxo5g4J0jEiaRsqibtFibn1A/640?wx_fmt=png)

当 var3.flush() 调用向 server 端发送数据，teamserver 的 authenticate(Socket var1, String var2, String var3) 方法就会停止阻塞，接受到相应的数据并对密码进行判断：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawFLdZQia4ndh80OxfH0pBkU63baH5e2NdyhxfwAmpNEjtZlhyCQnvxHw/640?wx_fmt=png)

若**密码相等**则发送 51966 标志数给 aggressor 端：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawZ13Y1ODia23ib77tIdS624gCPFYV2Tm4FeJtpxqYwjFWJWoklG2LntfQ/640?wx_fmt=png)

然后返回执行 PostAuthentication 对象实现的 clientAuthenticated():

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawuFpHuxlMxicJV0npng1BNwWl1iaibpIhwXSrCVQszFEibU7mnqHEDx6JSQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawG0FCZLiayLQjMIfmKD6Z4VPgolqHPpGvYBGicAmYxWicpognH9ZoUPKBg/640?wx_fmt=png)

新建线程，等待后续接受 aggressor 端所发送的信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw6OEsQXcXkZAXMlmvHjynWia20CuxnjRBtYicJow0BUaR7Rzia0Fpia0ibjA/640?wx_fmt=png)

aggressor 端接受到后进行判断：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawgiaNI4CAiadlHBcBb90AY8XMgibZfqnY4poYxxV2J0e2LmMHbkM7LD2kA/640?wx_fmt=png)

至此 teamserver 端校验 aggressor 端密码完毕。

**2.2****、****aggressor.authenticate** **消息类型传输：**

接着 Aggressor 端再通过 SecureSocket 对象构造出 TeamSocket 对象：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawOxVsCIjFtCxpq0p323iaibRic22PLNIaPjYLjw7nMgI94icOwz0NfMgQtA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawB0CgE2MomuESm8r270ngq1dUC7GkQeyYtp1J0xwYbrvA9lhHLWic1Vg/640?wx_fmt=png)

传入 TeamSocket 对象来创建 TeamQueue 对象，其中 TeamReader 与 TeamWriter 均为 TeamQueue 的内部类，均实现 Runnable 接口负责与 TeamServer 通信：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawUfh5C3GiantRTh0ic1rDicBfkwhl7hW66f9xXm5XraUwOsAN5kfE1EGrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaww8ic0GuK4mo4LgCQ8cucZSiaFwYK3NJaqWof3Zz6mAJqwp9Tob6f08xA/640?wx_fmt=png)

然后调用 TeamQueue 的 call() 方法，把消息类型 aggressor.authenticate、用户名、密码、版本信息均传入，再把请求通过 TeamWriter 属性发送给 teamserver 端：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawPJbJ1wyHHLDichbiaGNgcHQZ6cXbeIjwfpt6oHL5AsWRshqoWHapxXSw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawmqDEmysRAJo8gescrT5Uop1BmxGkibUXsy1kaYmd9yI3kuyr1hJe3XQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaweSukRKSjpWzuug6iaeib9yKKgCAu66eUDPkT204AfxOdlFe1m8fGdIUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw93G3RVGgSnYtadBPykAhDeXfiaw2PLNCT2GibrsvWQibJfvDsjn146kNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawlFTGZtXwLrwZcJBn3k9CkPe0ReVRcq2Wt5ceNY6DNWkEKXobNAAVKw/640?wx_fmt=png)

TeamServer 端收到 aggressor 的 aggressor.authenticate 类型请求数据包：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawgOyGvLUxgqfpD5zyGHKjFiaq0kbLHNzMZnGEpWMF4GFTMFDQ3wCU49A/640?wx_fmt=png)

跟入 process()，先比较数据包的类型，再判断版本号，接着校验密码和是否重复登录，最后向 aggressor 回复 success:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw4aYVWVCQ95Y82aeq1zctsicnpR0rKnkO4ustibnz3M6v5h7ghcmrBCdQ/640?wx_fmt=png)

最后新建名为 write for nickname 的线程向各个客户端回复消息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawN6tfFLvrvVyLysj2y0coaPBq7JAWdhc1OwcRuHRnibnxUwFvrdB7NvA/640?wx_fmt=png)

Aggressor 端收到 Success 回复：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawj8tgpPShHePiaU4fhV0IgZLlbSxqib5fyTUMZtFN9DVXaXH3foctA0LQ/640?wx_fmt=png)

**2.3****、****aggressor. metadata** **消息类型传输：**

Aggressor 端继续向 teamserver 端发送：aggressor.metadata 类型请求:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawvRcnXDzf9cL7TiafyFgTbd3oxZVJnGgXq3SKBeZ5dTYYXQcxktUzQ7A/640?wx_fmt=png)

TeamServer 端收到 aggressor.metadata 请求后，调用 process() 方法:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw1GdxlLZJhFwd1v8YuibYk7ic7FET6B2ricIcyEuazsC1JqibjicRjG684eg/640?wx_fmt=png)

继续跟入 process()，该函数主要用 hashmap 存放相关的数据，最后向 aggressor 端返回了 aggressor.matedata 类型和 hashmap 数据:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawTUEjkFNpwP0HSM9T903mLSOLfpmic0cmoQR9Km2fuktSIvIQOham5gg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawrhttddiahda5btPWed6wwGicPf4FVAJY4NCpLDSnFF7Y1kEEK1L654qQ/640?wx_fmt=png)

aggressor 客户端接收 teamserver 端返回的 aggressor.matedata 类型和 hashmap 数据:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9jLRuWAXzoCfL4Yd5eFL6MHIGAS1IkaicC38icIKicrvibKp09fWn2iafCQ/640?wx_fmt=png)

跟入 processread() 函数，函数主要获取消息包中数据类型和数据:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawwlUROqt8jauujByiczKKCe9BWJRDUe4FtrtOia7Ul7Rt3rRFaSX82zUg/640?wx_fmt=png)

再跟入 result()，可见该函数负责处理从 teamserver 端返回的 aggressor.authenticate 和 aggressor.metadata 类型的信息:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawMHGxN5uWbAbpMuHsW6icUqDEqHwOSDpTPTXxwia5Abib7NtQlWzZwaicKw/640?wx_fmt=png)

传入 metadata 数据 hashmap var2 来构造 AggressorClient 对象，跟入该对象的构造方法：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw06ibuoxEKtOFlfDPAU5OOicxVm6T0rLp4ZYQ7h8L2jTlRibVzsg9Zct1Q/640?wx_fmt=png)

继续跟入 setup() 方法，该方法主要是注册各类 Bridge 对象，储存 teamserver 端返回的 metadata，加载插件框，以及发送 aggressor.ready 给 teamserver 告知其准备完。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawhPtNibc5m0dWU5FY65WrH7CDpicfc7JG7iblUKPCr9vjbJrKCe6tiaU8SQ/640?wx_fmt=png)

最后调用该对象的 showTime()，至此显示出主界面，后续同步相关数据:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawGzFicqEPceHgJz6aqugxdGticGibhHxicWAN0U0qTX8JdNKhooSrLIicBuA/640?wx_fmt=png)

下面图是 360 空间测绘的，顺手帮他们改了图中的小错误：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawfNU1LmGNRvmw0fycCCl8pmV9ve3vICqobyr4MDRteYkGZq4AQ61KSw/640?wx_fmt=jpeg)

至此，我们知道了 cs 的 agressor 端登录 teamserver 端通讯方式，所以可以进行相应简单的修改来规避 cs 登录爆破：

teamserver 端避免使用默认的端口号 50050。

如果修改 48879 这个标识数，需要修改的地方有三个，涉及到修改 aggressor 端：

SecureSocket 的 authenticate 方法：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawGXx8HOFLNlAadOUrL20sBiceVUqQUfFTJgu13NqASJATLxgqSPsxUqw/640?wx_fmt=png)

AsymmetricCrypto 的 decrypt() 方法：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawRXXxUnErMxBVuNX1RMKkPaBqLuYYlDOh9Wadp3GwNKe0piae4fma4dg/640?wx_fmt=png)

SecureServerSocket 的 authenticate 方法：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawY2j5JicbbtvTY7P1JttnJicqLeDNyIj21PtIvOsOAk46AFcUkibSibiaqGw/640?wx_fmt=png)

3

beacon 端与 teamserver 端第一次数据包通信（元数据）

我们先生成不分段的后门 beacon.exe，因为不分段的 beacon 更好调，其通信过程都一样的。

当启动 teamserver 后，会把 cobaltstrike.beacon_keys 反序列化为 Keyparis 对象：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw7dVibXAfgqibRbSnCJwwru6SyMS3C0ApD1uWy0ebj4HnGU9wL7d8C0DA/640?wx_fmt=png)

然后把该对象传入 AsymmetricCrypto 构造函数中，并把 publickey 和 privatekey 保存为 AsymmetricCrypto 对象中相应的属性：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawCU3o3Ocgzib1Yb6u9P15CwcXGpWiaclXz64oBdo233d1YpiatavFHxCVQ/640?wx_fmt=png)

所反序列化出来的公私钥：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawN0YGBWZId2klOFRoEp9T58ClSZB06DXLh7Aqm8EX4FJUkX06an2kAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawHgHBRnicKmPyCovCYNtibrXxP7S7XU9UyJGnYcqF8gHsTZuRxot1UVnQ/640?wx_fmt=png)

公钥会在后门生成时放进去，但不是在上述的函数中，详见之前发的后门生成的帖子。而私钥则是为后续通信做加解密处理准备的。

下面我们继续操作：先直接上 x32dbg，createthread() 下断，查看线程调转的地方再相应下断，然后执行到反射 dll，再等所导出的解析函数解析完 dll 后，再通过导入表搜相关的通信函数，因为这次文章主要是协议分析，相关中间的相关功能先省略掉，有必要再从栈回溯查看相关的函数调用，然后我们看到相关的通信函数：winnet 的 HttpOpenRequestA、HttpSendRequestA、InternetCloseHandle、InternetopenA 等，还有 ws2_32 的 socket、send、recv、accept 等，通过观察 ws2_32 用的是 select 网络模型，则先排除掉：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawv05IottN9euM85Ik8KY2HvWlicUTBAo8g2BdkibDh83hyTDTucSoyWCg/640?wx_fmt=png)

所以从 winnet 的相关 api 下断入手，

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw0D4EWibXv2EZic2RUbO0c7L0YlJW74scx22AGOic8F9kqULfSzmibJjF4w/640?wx_fmt=png)

程序先在 InternetOpen() 函数断下，传入相关的 agent 信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw3iaMVfntxTv0gJMLNp0rxyxrJhre2GKoso6d4pADMCA9f3rlCbv10hw/640?wx_fmt=png)

接着 InternetConnect() 函数，传入相关的 url:192.168.202.1 和 port:80

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawUtM7zbbAibyG7aicicmfWJH5VKH0NAT7EKgjt3iac7Nk5OzWXYlNkJDz0Q/640?wx_fmt=png)

httpopenRequestA() 函数，get 请求，uri 为 /pixel.gif

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawtq9r5L732ZffAAtr0MZQucW04pdIB9Rz7melO9DVo4eAwBS8ib5DOGw/640?wx_fmt=png)

httpSendRequest() 发送请求，注意 cookie 的值经过编码：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawK6AtdBv9v1jggYYKAe6fWHtadakPphzkjsiaKp5pghjPZVVpauiaubsg/640?wx_fmt=png)

Wireshark 所捕捉到的数据包：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawQXBM9fRZVa1r5TI56LLr5IMSlLLiaNTcMbnP69fxeBiadAfkzBicDW36Q/640?wx_fmt=png)

可见 cookie 对应的数据经过编码或加密的，我们往上翻找到相应的函数 sub_1000783E()，跟进该函数，可见函数有 16 个不同的 case，当发送第一个数据包（元数据）时先后走 case7 和 case3，case7 只是复制操作：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawc2ZydvPKdDMoLrkE7TrKwxfiaB6A6MZpSTwXzicj9OvKu7VtX9aN2bJg/640?wx_fmt=png)

Case3 是自实现的 base64：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawUXiaRc3wwjOAxHIUVs8hQlqwSosKQiafr6L1DKiadVHJ1HOsTMfA26mfQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawwPyLy7aLVQcHNm0uFYcW1VLeCnr3O34ictAou7ssGkctrRibYBbHviazg/640?wx_fmt=png)

根据所编码的内容 &unk_100398A8 继续找相应的赋值和加密操作 sub_1000671C()，在该函数中，收集受害者主机的代码页、时间戳、程序 id 等信息并生成 hash 值：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9KtLib5Chj6ewSzyicibseCNISkyZX6IwrjRnzY0ucicUeyaGuljbVrNpg/640?wx_fmt=png)

生成 hash:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawTLmOfybUOWON7ib22oNtr9L9zaiafE27cmtlAvoGbJiaok5Z5ib2Db8QMA/640?wx_fmt=png)

最后调用 sub_1000969e() 函数，对收集的信息进行 rsa 加密：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawKTJkHdK2JztIibz64ukhc2u1icMetSgqE2ictWicJ8CJTQZnbc7r31HMQg/640?wx_fmt=png)

用来加密的 RSA 公钥：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawJsp1crPJaMsRAYcPpyKPUj24mh4RE5hTJqjSWiarzpWibSVJILT26ibyw/640?wx_fmt=png)

RSA 加密前所收集的数据内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawkqjGEf3oKBNWBOmEjq0Ulb6IRCx2bqXpEVtdWWAquaI3fkECY5Hayg/640?wx_fmt=png)

被 RSA 加密后的数据内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawVfflHrrh70EeDdjELCz0dnVvOh0cLqaNGsA0t6picWeqJfptCM4uryw/640?wx_fmt=png)

至此，beacon.exe 端第一次发包（元数据）分析完，流程总结为以下：

**收集数据** **->RSA** **加密** **->base64** **编码**

下面开始分析 Teamserver 端接受 beacon.exe 元数据的请求：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawfNibR1U08tD4qf8519tkLep7UsNVGmVTvnibLCC6jEoHQ02SG7yiblk8g/640?wx_fmt=png)

跟入_server()，该函数经过 Useragent 等判断后，先把 url、method 等相关参数传入 webservice.serve() 进行构造相关响应头并返回，其中 MalleableHook 实现了相关的 webservice 接口，跟入 MalleableHook.serve()：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawKQZqCic0uEdwflUrtAgyLoZMyianZIWiaWvDogRtpkD3sJl0vKO54yYpA/640?wx_fmt=png)

在该方法中调用了 GetHandler.serve() 并把相应参数传入，继续跟入，该函数中先解析出相应的 URI、method 等：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawwsGd4bvS8609ic2icIiaib5dAJANdc3hCAAKJibGDc0AybOkfIiblSic1dqEQ/640?wx_fmt=png)

我们先跟入 BeaconHTTP.this.c2profile.recover() 函数，在该函数中先将 cookie 数据拿出来：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawibwvDsN6aEmibOC1PAia5pqFnqeCedBrvrBJcgZ12Ad5ZEh2BqMdic4LRQ/640?wx_fmt=png)

然后再进行 base64 解码：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawx2M6dzzWL9iciaz6oXpt3BxjZFiaUfB12aefwaQ07nwb7O91MtYQprqQg/640?wx_fmt=png)

解码完成后，会判断解码后数据的长度，接着传入被加密的数据到 process_beacon_metadata() 进行解析，继续跟入：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawu6B28YHia68D01oYXLrdgRLImfTL7dTXu3vfib7dDwtlv7xRV4tCaUyQ/640?wx_fmt=png)

跟入 decrypt()，首先把反序列化处 RSA 私钥，然后对原数据解密，接着判断标志数是否为 48879，如果不相等则解密失败，然后再判断数据长度，如果大于 117 则失败：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawqISu9KxWdYVCqQ2vHqgEFHpmT5zIUNCweeGsseKWSKStDO5SLUIObw/640?wx_fmt=png)

回到 process_beacon_metadata()，解密完后先读取前 16 字节备用，然后用 16-20 字节判断 windowscharts：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw5VicYHKcUdsWNPUjtG61zwibBmzNaN1ZX4qibmRToKiamf6Voc0nvMlibibg/640?wx_fmt=png)

把 ip、windowscharts、listen 地址、截取后的元数据传入到 BeaconEntry() 的构造方法进行解析：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw0yAzr0rNIRtU5E6Q9NTDlRgmUibEytfZVyr8391mtibgLsiag7Xw3L3KA/640?wx_fmt=png)

跟入该方法，方法中先跳过前 20 字节，然后获取到 id、pid、port、系统位数、受害者 ip 等信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawDGBqPvUeRJiaibbP5PzTfo5n195gA6rfuR8tCcPXRp8IUHzd237pX0kg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw17cM71N6UyNAfgZyxeUVFULzdJ1QQCI7bzdCk6TNGnDfldTiaHI6XEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawDbWjDul9QG2nicUiagn79QibA6320Um1XOAW142Cbs0KPEDS69OuhlDDQ/640?wx_fmt=png)

构造完后返回到 process_beacon_metadata() 函数，再传入元数据中 hash 值给 registerkey() 该函数是注册 AESkey 和 HmacSHA256key 的，对传入的 hash 值进行一次 sha256，然后前一半作为 AESkey，后一半作为 hmacSHA256key：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawNdbcMdoeicuRgrSJUdqkW9UWP3dwo7SogSEGqp705LRiaO8EYzSkWLYg/640?wx_fmt=png)

最后构造 response 数据包返回给 beacon.exe:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw8Yeic5pEjrMp9Rp5PeN3aTS5j0ybl1Df0QUoQY3qicib9XaxLIeSHGpXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw1WNXqPFoqeU5B9icV6UrVibgOJKaALq2ArVqjaznNET0ogicyknEOGCDQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawpbhvGr7klwSt1zsoCv4BicNrHnt015Bej8CUnxhicK8TpeMtWzG1gyQA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawxQldoIrmichn9cVfrt6cSpnUF06MibCcY2rfJdaBv6g6k9ksEnIqk03g/640?wx_fmt=png)

4

beacon 端与 teamserver 端心跳包通信：

心跳包和发送元数据包的流程是相一致的，都是先复制加密后的数据再 base64 编码最后拼接 cookie，只是没有再走 sub_1000671C() 函数获取元数据了，只是走 sub_100024F4() 发送数据，数据的内容也是与元数据相一致的。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw4IoujMQVJfOLA4ibVWciaOI7lGWjOyx9MBJicz39f4qGSsnAnMZJFeWWA/640?wx_fmt=png)

5

beacon 端与 teamserver 端执行所需任务的通信

下面介绍 Beacon.exe 接受 teamserver 端的任务数据传输：

**5.1****、****teamserver** **端怎么发送任务给** **beacon** **的？**

当每次心跳结束后，teamserver 端都会把相应的任务加入响应包中发送给 Beacon.exe

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9UmY0oZ2uarXpkpwhtmscDsKCcNr1CmZKhGvsGQ2GkJXw9c8hyjb5Q/640?wx_fmt=png)

首先会根据 beaconEntry 的 id 号获取到任务队列中的 data 和 socks：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawlwHLWkHUrfXWAsdvUeEzZwRDAfO022AwicgEu0qfHOlyc1sDq6VwVxg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawgMaYyLCpNxlvOCx9tRGT3J4bwukE98mXNwtsXjG0rzxDQAVU1a52Gg/640?wx_fmt=png)

然后把 beaconEntry 的 id 号和 data 数据传入 encrypt() 函数进行加密，跟入 encrypt() 函数，函数中先通过 beaconEntry 的 id 号拿到 AESkey 和 hmackey：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawiaXyZuBFtaeicJkZcyWnC86fN152lORz0fhndSahcwQcfyu5ibAAxPGGQ/640?wx_fmt=png)

获取当前系统时间 /1000、data 长度、data 数据并将其写入 var3，然后进行 aes 加密，密钥为 beaconenrty 中保存好的:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9czKWd2A1wj0g3QSB4eFFBFfthtIdyjpcPIJZ1IyOKxm9A63AdjHsg/640?wx_fmt=png)

对其加密后的数据进行 hmac：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawkhEibGSCL9KfS06VL2TaiaJzO1PU8yUzxFtBnsPtIc4pQ6pApoQjc3rQ/640?wx_fmt=png)

将 hmac 前 16 字节拼到加密后的数据后，并返回：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw4Bm2kMicYrBiaFV9T43ez9yq3TVOYJ9y1dSmVnakxxF4xteaYM11SMWw/640?wx_fmt=png)

最后发送给 beacon.exe：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawzj9LkXIzibTicqBaFUQGmZckwjIEpicybLXP0iaTysMcqrK7tyQJTU0rZg/640?wx_fmt=png)

**beacon** **怎么处理** **teamserver** **端的响应包？**

InternetReadFile() 读取到响应的内容

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9G9UK5vj5AVwMB3woV5aVPJfIUSMdPbydCQYcAjl2hIlF4oqWSBdZQ/640?wx_fmt=png)

然后在 sub_1000c901 中进行相关的自己实现的 sha256 验证：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawyF2TBFfqqI7egDe7hiafGOe2jYLdP4Me3odOlrNBjWnrn2yR2exGs6w/640?wx_fmt=png)

当验证通过后调用 sub_1000c5b1 进行解密，我们来看下解密的流程，显而易见这是一个 aes_cbc 解密的流程，先解密再异或，v4 是加密的内容，v18 是解密后存储空间，后面是 context：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9TddcSYP06JVQibozhysBV7HVn0dxV91DwRgra73lrj7plyYdjf8Tiag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawg9VOGUHsYfw1FLPTVhJj6DZlymY0LSYtlTsGyoNAUQvJKib1wl5F6Zw/640?wx_fmt=png)

Context 内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawzZbia98xbLUutkoyDcNzZuiaPNc8yh4Njibrf5XRFZic9ZOCHaBwjYEewA/640?wx_fmt=png)

解密后内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawLQHPq8yj4k9v0zzG23fwn9VjUF8ibFatByuJr8Wxb9NIf3YiaVLl3CkQ/640?wx_fmt=png)

Iv 为 abcdefghijklmnop：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawIrt3B6HmeLMic9t0tPqsFcV5UAf8FOmzuhU8wiblichp1d0t72oW2Eb0A/640?wx_fmt=png)

解密后的内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawG804DlNGqpAbHwic6PbwAbYJ9ZKT85RVozP1uzgGzKl7VwZeJedgSbg/640?wx_fmt=png)

后面解析解密后的数据，先获得时间戳，再获得数据长度，最后获得数据内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawmUORPicvHtbw9ch2ic9Svc3tSD2cxvq8hv8BIaX9PCPzLzRJvAQ4uA9Q/640?wx_fmt=png)

**5.3****、****beacon** **执行完任务怎么发送数据给** **teamserver** **端？**

当 beacon 执行完相关任务后，调用 sub_10001345() 函数把数据内容加密:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawkXSfu5KlWXgAHxgfIISNPum7Pt826ct8HPY66mou8YOo1u8mUdZTkQ/640?wx_fmt=png)

然后经过长度和数据类型等拼接后，调用 1000C170() 也是进行 AES_CBC 的加密：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawjF1F1sxW9284bjqruuDN62Mf36PSSLNIeCP1zIpwAbsenq3FOBQ7vQ/640?wx_fmt=png)

最后把加密完的数据调用 1000C901() 进行 sha256 的校验，校验值补在后面：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawVWwRXCia75icpoibyymLAaH3dWDSdiaEPe77zh5x3yLaEbic1VkR2vk3CTA/640?wx_fmt=png)

发送数据:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawib8EB2u7kZmqhphtYSZxmfVVRKHXjYUcbBVKx3BeuZS8j5yhCGU48kw/640?wx_fmt=png)

**5.4****、****teamserver** **端怎么处理** **beacon** **端执行后返回的数据？**

接收数据的方式与元数据一致的，先在 BeaconHTTP.server() 获得 beacon 和 id 和 post 数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawk5xX1LOjobAvic2dNdF7SCPAJSojAaeWRjosiaCF9OIs7VjHJZqvwx1Q/640?wx_fmt=png)

然后把 id 和数据传入 process_beacon_data() 处理，在该函数中调用 process_beacon_callback()：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw0PECJZnib0WtbR6TictTibeibzLWm2l33NKESL385XZpFfsXibFA9ImNgWw/640?wx_fmt=png)

调用 Decrypt() 来解密数据，解密流程也是先对除了后面 16 的数据进行 sha256 验证，验证通过过再进行解密：

再把![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawkHjVulo5A4rpW9tiadGOpZzIBBBdVgGVlNL9A9oBjqwn4tFuibYDOgzg/640?wx_fmt=png)解密出来的数据传入 process_beacon_callback_decrypted() 函数进行解析：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicawvnKt1gPjiacqWMRwSj4eqLHavBkMhvibQk9dc50NdyKibRm7Pr4Id7s4A/640?wx_fmt=png)

6

关于流量检测

这次我们一起剖析了 CobalStrike 的通信协议，可见其通信协议是可靠的，RSA 传输 AES 的密钥，AES 的密钥加密后续通信，这也是 c2 的较常规通信手法了。如果在配置好 profile 和证书的情况下，还是被检测到流量通信，不妨一试删除默认的.cobaltstrike.beacon_keys，

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABbpsT2aviabQvj0owCSRicaw9rtzEN7WXhohDic7qloo0ZNNuzQcumBxSeMs7un25LsCaOFp7oUHyicg/640?wx_fmt=png)

这个文件是可以删掉的，删掉后会重新生成新的，但改文件保管着 RSA 的公私钥，所以要注意的是一旦删掉后原来的链接会丢失。也可以使用 ExternalC2、AWS Lamdba CS Redirector 来规避。

7

关注  

关注本公众号 不定期更新文章和视频

欢迎前来关注

![](https://mmbiz.qpic.cn/mmbiz_jpg/PrTu58FA79bYUuGICO85hGrTyicvB3nMAtd7QY3C0H3CA2SOwaiaSkDbazCO8C1VXHx8ticGRxDeVATd9LZf62z4w/640?wx_fmt=jpeg)