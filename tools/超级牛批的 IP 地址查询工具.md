> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kTUOJf4Jqik9sP4g_g721Q)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fuBhZCW25hNtiawibXa6jdibJO1LiaaYSDECImNTbFbhRx4BTAibjAv1wDBA/640?wx_fmt=png)

扫码领资料

获黑客教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFJNibV2baHRo8G34MZhFD1sjTz4LHLiaKG9208VTU6pdTIEpC9jlW6UVfhIb9rHorCvvMsdiaya4T6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png)

Fav-up
------

Fav-up 是一款功能强大的 IP 查询工具，该工具可以通过 Shodan 和 Favicon（网站图标）来帮助研究人员查询目标服务或设备的真实 IP 地址。
--------------------------------------------------------------------------------

工具安装
----

首先，该工具需要本地设备安装并部署好 Python 3 环境。然后广大研究人员需要使用下列命令将该项目源码克隆至本地：

```
git clone https://github.com/pielco11/fav-up.git

```

接下来， 运行下列命令安装好 Fav-up 所需的依赖组件：

```
pip3 install -r requirements.txt

```

除此之外，你还需要一个 Shodan API 密钥！

工具使用
----

### 命令行接口

首先，你需要确定如何传递你的 API 密钥：

-k 或—key：向 stdin 传递密钥；

-kf 或—key-file：传递获取密钥的目标文件名；

-sc 或—shodan-cli：从 Shodan 命令行接口获取密钥；

配置好密钥之后，我们就能够以下列几种不同方式使用 Fav-up 了：

-f 或—favicon-file：在本地存储的需要查询的 Favicon 网站图标文件；

-fu 或—favicon-url：无需在本地存储 Favicon 网站图标，但是需要知道目标图标的实际 URL 地址；

-w 或—web：如果你不知道 Favicon 网站图标的实际 URL，可以直接传递目标站点地址；

-fh 或—favicon-hash：在全网搜索 Favicon 网站图标哈希；

你可以指定包含了 Favicon 网站图标的 URL 和域名的输入文件，或者直接提供 Favicon 网站图标的本地存储路径：

-fl 或—favicon-list：文件包含所有待查询 Favicon 网站图标的完整路径；

-ul 或—url-list：文件包含所有待查询 Favicon 网站图标的完整 URL 地址；

-wl 或—web-list：

当然了，你也可以将搜索结果存储至一个 CSV/JSON 文件中：

-o 或—output：指定数据输出文件和格式，比如说 csv，它会将存储结果存储至一个 CSV 文件中；

工具使用样例
------

### Favicon-file

```
python3 favUp.py --favicon-file favicon.ico -sc

```

### Favicon-url

```
python3 favUp.py --favicon-url https://domain.behind.cloudflare/assets/favicon.ico -sc

```

### Web

```
python3 favUp.py --web domain.behind.cloudflare -sc

```

作为模块导入使用
--------

```
from favUp import FavUp

f = FavUp()          

f.shodanCLI = True

f.web = "domain.behind.cloudflare"

f.show = True

f.run()

for result in f.faviconsList:

    print(f"Real-IP: {result['found_ips']}")

print(f"Hash: {result['favhash']}")

```

所有属性
----

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibuaeUhVpt9VooEeFGuiaKwnicHzMpuhUicT0K7VPIjNU6bYQgfmtevm4jqKiaFafRjiaRGx9HETTHssHQ/640?wx_fmt=png)

工具使用截图
------

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibuaeUhVpt9VooEeFGuiaKwn0hBVXMyk2SrthTySvdtqzEFg6sOvmcBWAwhz4PCvRjDtGR65ESRMKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibuaeUhVpt9VooEeFGuiaKwnU52QibxXFct2bENkIfpNDVFrMKBBXSILtLIvNtJjAn17bOArfuVc7NQ/640?wx_fmt=png)

项目地址  

-------

**Fav-up：**

https://github.com/pielco11/fav-up

@

**欢迎加我微信：zkaq99、**实时分享安全动态

* * *

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSE8r6UDibLl3oFOu6cEZPryVrS6n7TfhmDVMfKfIfc7nicyXQ0r0CjPZxPIACeen4QF4fuLwsRBhzMw/640?wx_fmt=jpeg)

[

实战 | 渗透摄像头

2021-06-07

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSH9t7u1E1mfrIxADDU04GcKFLnQuiar2DRV6ul6WicaAx894LSL9XKPeoOptKONXEpufUvRfHEqj8IA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521870&idx=1&sn=a4201fa9fa2fa7ac0693c7e324953993&chksm=ebeadf63dc9d567522f4d32b0ae33e03edf0964de0124bd53cc45b495d3fae8417eebfb732c4&scene=21#wechat_redirect)

[

记一次批量刷漏洞

2021-06-11

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFyVfgq0XrOk4T4z5stANyHQCUlS80H0GWrU59uh2iafdpRRNtse6Wa9Cok0ntpWbMYJ9qBcRRXq2g/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247522606&idx=1&sn=d7b52504576d3d4450979ea14158d3ea&chksm=ebead003dc9d59150c4c1599391975db3bf41d5267f270947819acbef539fa392f05c5ce2ad8&scene=21#wechat_redirect)

[

利用 python 完成大学刷课（从 0 到完成的思路）

2021-06-06

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHkvnCnsBn2mKH5btCeBEkhOWBF94KdmIDM01G5bUUvibhG4KMw5f84BZvwWXibyxYXqTHiaduyCuVrw/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521848&idx=1&sn=ea6348b0b3978f4b3814a1c6a8b86a46&chksm=ebeadf15dc9d5603f947d79880e13aec69778a464f69954998ed6ffe895a2f5c78cb57ab01f9&scene=21#wechat_redirect)

[

黑客技能｜断网攻击与监听演示

2021-06-04

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSE8YF0okRDl7zWnKCPoxDGUZeEaKAuibz1Wiaj3iaJJic8uoD1bVPIUv1hFKL5b1iauiclwiapBmAibEtjJEA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521713&idx=1&sn=ae4efd5b60465cf44be7b26e9abb5579&chksm=ebeadc9cdc9d558ad2b3dadf55a5571a0a5a52453248069186b2dc30101d9fc7b6cd5b88b343&scene=21#wechat_redirect)

[

实战 | 一个很奇怪的支付漏洞

2021-06-01

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFbkJ5sialXP7Ab8JVxBfCQqicAjFhXjUibpB1GR9AAkolWMXhoZwa6RBtEvJV2e77lhKG8QIGIO9wicA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521470&idx=1&sn=dfe9c0aec50019b0922d8c5b844883c8&chksm=ebeadd93dc9d5485a58290d4b77f4c62520d3ada70cef2c6b3698dffd8e2a120f95ec42f6c57&scene=21#wechat_redirect)

[

暴力破解工具—九头蛇（hydra）使用详解及实战

2021-05-29

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSE8YF0okRDl7zWnKCPoxDGU6zSh8GYRFdyCwk2JibfaNqJlhMqdkh9XiaNr9doiatbg796eFvcSKINBg/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521292&idx=1&sn=d41793e5138353a61c1c773f612c6d30&chksm=ebeadd21dc9d54373a3a08ed67cbe551a2d0dc2ea3414ca10f449ec9f3197ea13714165908f4&scene=21#wechat_redirect)

[实战 | 一口气锤了 4 个卖吃鸡外挂平台](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521247&idx=1&sn=7fa81e0ef16f2ca8088c948c2b0ce1a7&chksm=ebeadaf2dc9d53e49c681a4f61adc5071e8833ee37d4f32191a5a0fcb4b827ea0736a4407bed&scene=21#wechat_redirect)

[2021-05-28](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521247&idx=1&sn=7fa81e0ef16f2ca8088c948c2b0ce1a7&chksm=ebeadaf2dc9d53e49c681a4f61adc5071e8833ee37d4f32191a5a0fcb4b827ea0736a4407bed&scene=21#wechat_redirect)