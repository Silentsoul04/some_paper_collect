> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aEC6FI8Ksyafgsd97mu2_Q)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icD8Hl3Ea5fHmlzicxWkh4b9hrmesI8RfiaTLzv0LzyGibRXibMPbQmFpico8g/640?wx_fmt=jpeg)

关于 PwnLnX
---------

PwnLnX 是一款功能强大的高级多线程、多客户端 Python 反向 Shell，可以帮助广大研究人员对 Linux 操作系统进行渗透测试。

**请广大用户合法合理使用这个反向 Shell，请不要在未授权系统上使用该工具。**

工具要求
----

首先，我们需要在本地设备上安装并配置好 Python3 环境，然后安装下列模块：

> vidstream
> 
> pyfiglet
> 
> tqdm
> 
> mss
> 
> termcolor
> 
> pyautogui
> 
> pyinstaller
> 
> pip3
> 
> pynput

工具安装
----

使用下列命令下载 PwnLnX 源码：

```
git clone https://github.com/spectertraww/PwnLnX.git

cd PwnLnX
```

接下来，下载并安装相应的依赖组件：

```
chmod +x setup.sh

./setup.sh
```

PwnLnX 运行
---------

显示帮助信息：

```
python3 PwnLnX.py --help
```

监听传入的连接：

```
python3 PwnLnX.py --lhost [your localhost ip address] --lport [free port for listening incoming connections]
```

创建 / 生成 Payload：

```
chmod +x PwnGen.sh

./PwnGen.sh
```

按照操作步骤，我们就可以成功创建好 Payload 了，Payload 存储在 PwnLnX 目录中，然后我们需要将创建好的 Payload 发送给目标用户。

PwnLnX 使用
---------

<table><thead><tr><th><strong>命令</strong></th><th><strong>使用</strong></th></tr></thead><tbody><tr><td>help</td><td>显示帮助信息</td></tr><tr><td>exit</td><td>关闭所有的会话并退出程序</td></tr><tr><td>show sessions</td><td>显示所有可用的会话</td></tr><tr><td>session [ID]</td><td>使用指定的会话 ID 进行交互</td></tr><tr><td>kill [all/ID]</td><td>终止指定的会话或终止所有会话</td></tr><tr><td>banner</td><td>修改程序 Banner</td></tr></tbody></table>

### 与会话进行交互

<table><thead><tr><th><strong>命令</strong></th><th><strong>使用</strong></th></tr></thead><tbody><tr><td>help</td><td>显示帮助信息</td></tr><tr><td>quit</td><td>关闭当前会话</td></tr><tr><td>background</td><td>当前会话后台运行</td></tr><tr><td>sysinfo</td><td>获取目标操作系统信息</td></tr><tr><td>create_persist</td><td>创建一个持久化后门</td></tr><tr><td>upload</td><td>上传指定文件至目标系统</td></tr><tr><td>download</td><td>从目标系统下载指定文件</td></tr><tr><td>screenshot</td><td>获取目标系统桌面截图</td></tr><tr><td>start_screenshare</td><td>开始桌面共享</td></tr><tr><td>stop_screenshare</td><td>关闭桌面共享</td></tr><tr><td>start_keycap</td><td>开始捕捉目标用户键盘记录</td></tr><tr><td>dump_keycap</td><td>导出 / 获取捕捉到的键盘记录</td></tr><tr><td>stop_keycap</td><td>停止键盘记录</td></tr></tbody></table>

注意：除了上表中列出的命令之外，我们还可以执行各种 Linux 系统命令。

工具运行截图
------

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDW8icOTbYVNqGP9DSxibDZCBiaKG9JotZa0ibUvmWn1DWDpSzvEZxHraiaQA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDgBRbq6xxCMSDSufg3VItEwZODqNrqDZzZoJVpnDO67gibsu3Uicgjlibw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDueibbCEcTrzQJ6rRwJHMQU03sCLfPjyzln3JvXibTvu4FnZyLe3m4aUQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDwicePFK1U8XeZIicl6zqgpenOOw4N6Ev7jTRHHcAjscqsQVS2xSMvtyw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icD7q1CicticoRvcica3XXRlVs8BrZKxSXRpRR1yzv4ECIlia4FojvFiaKGgibw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icD7q1CicticoRvcica3XXRlVs8BrZKxSXRpRR1yzv4ECIlia4FojvFiaKGgibw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDiblGogr1SjPS1gSRgZQBlpnGfP5E8ib4gEqTCM1s0bBbMNXK2THRPEsg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDk2X5qfcHp1e2QezC5HIhUXyey8tgslqkDvROuDEWNdCD57lAoyyCVQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib77P8Jzu2YGSH9kxlZj4icDZLfJibLATxMmvpb7dQAiaWnZAjGmdAVjzqeWz2Qhyoty3YYiaWibr3Y79g/640?wx_fmt=jpeg)

项目地址：点击底部【阅读原文】获取  

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

  

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38seEkNn8TH7jZibkFTmoEsk6RKElsJrrsciaM7x32aqsPkBRK96QbqftgV9wWoG4HzVibedTiaZffTcg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651124157&idx=2&sn=f122abf33374d9bb2d105bb7afa93c74&chksm=bd1f63368a68ea20963d042cbc7e65e763a169b27f9cab26b3ce940603ac7fe8976260701a35&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39dEsdO2GpOvH87GrfzuscAMuA4JpicWAFbJtfakgMF2hheeTcSSwguAbjO45btx8ws2etnvSJlOzQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486070&idx=1&sn=c6957ca2d1878f316b7947b5ff990a01&chksm=ce1cf0e9f96b79fff5b27a3c146f9e8828728c33625a97366b0cae3df1853dbeda368c59177f&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39n5GEibfNkw4IJCQ3PU5W4hScYnG2TeOSgTVGYX9BZfoBX4cvliaEolz3gepYFfNvlFMYvibbmn0Rzg/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486050&idx=1&sn=7e7d54cc1319f1dadfd36b4f92974c62&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************