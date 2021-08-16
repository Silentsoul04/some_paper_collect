> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Wh-A3qiyrjPv0KzYeAS-Xw)

![](https://mmbiz.qpic.cn/mmbiz_jpg/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxkPBC14zmGlW1k98taNIDXCWoCQLAcVxoZKobhTDCVibfGzsRS4DXIicw/640?wx_fmt=jpeg)

目录：  
一、事件起源：对一个钓鱼附件的分析  
二、域前置的相关配置  
三、总结  
四、参考链接

### 一、事件起源  

对一个钓鱼附件的分析：

在近期的相关活动中，我们针对收到的一个钓鱼邮件中的邮件附件（本文中暂用 “钓鱼. exe” 统一代替）做了比较详细的动静态分析，以下是我们使用某沙箱对 “钓鱼. exe” 自动化分析后得到的运行流程图：  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxWdmsLGjQ12kA9NWysib5WfUZfibhFYak5KZPaMZGdZ4rEIibvmRaiaVMIg/640?wx_fmt=png)

由于 “钓鱼. exe” 在回连 IP 时使用的是 https 协议，我们对其联网活动行为进行了解密追踪，发现了一个比较有意思的域名“www.windowsupdate.com”：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxryuthIxM4tt4cib8pggeTK4HFjBsoD8SWNRfriaiaL89k5ZXrUQCTjP1w/640?wx_fmt=png)

以上的所有手法是 “域前置技术隐藏 C2”，在蓝队的日常工作中针对这类隐藏攻击者真实 C2 地址的技术手段是比较难以溯源到真实 IP 的或者对抗成本较高，在实战攻防中也是红队队员比较常用的一种逃避全流量监测设备的技术手段。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxRzI7WhH5ESRO6fLWSD3LO4oavUnknC1KEuojSPTyGlcJADvZtEtpiaw/640?wx_fmt=png)

鉴于现在有的 CDN 运营商和云服务器厂商对新增申请的高仿冒、高信誉域名的归属权认证有比较严格的限制，目前在互联网对 “域前置技术” 的介绍文章使用的方法大多都失效了，经过一段时间的尝试与踩坑就有了今天的这篇文章。

### 二、域前置的相关配置  

步骤一，对 CDN 的配置：

在某云上申请 CDN 资源，测试域名填写 “micrsoft.com”，加速节点类型可选择 “网页”

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxEiapU5iaI7fSBiarE2ZugnFBHFZ0tgjkrj89BON168byD1xdbS7pED9kg/640?wx_fmt=png)

在创建域名记录时可以自定二级域名名称，本次测试使用 “qqqqqq.micrsoft.com”，回源地址填写自用的 CS 服务器地址：“111.x.x.86”

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxpCIlpEnQUbkX1QvCru4nAXM5FZmV75B0jkTXXyPnzP7DfKUD2S1GnA/640?wx_fmt=png)

等候 3-5 分钟后 CDN 状态变为可用，需要记录下分配到的 CNAME 地址：“***.cname.frontwize.com”，其他配置默认可不需要调整。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7Ox6aW6asGxgZicGnhHFthicL2RlJDbUZOffRia1al2G4Rd6dOhsialsS91YQ/640?wx_fmt=png)

使用站长之家工具：ping 检测（http://ping.chinaz.com/）对分配到的 CNAME 地址进行测速，可根据实际情况选择延时较低的几个 ip 留作后用：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxSmy3AluGFrX8XdxqP8dCnFpU6oTrpu4fz9CLfA0O30hrp7QrIeB02w/640?wx_fmt=png)

步骤二，对 Cobalt Strike 服务端的配置：

修改 Cobalt Strike 服务端的 profile 文件，我是基于 amazon.profile 文件的基础上修改了两处 host 项，均修改为 “qqqqq.micrsoft.com”

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxhEmicNQ9os1ykpMC1S5Wf4OvjYQnBiaX6jcPOicX8I0OxicxOo9F9VURDw/640?wx_fmt=png)

保存修改，命令行启动 Cobalt Strike 并加载 profile：

./teamserver 111.x.x.86 mima amazon.profile

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxVwLfVjFmS8zSptVzgibQIDUl2NR36hvcEx8ndmU9aUJztorpXonRJfA/640?wx_fmt=png)

新建监听，HTTP Hosts 中的 ip 地址填写之前经过 ping 检测后的低延时的 ip 地址，HTTP Host(Stager) 一项填写 CDN 加速测试域名 “qqqqq.micrsoft.com”，监听端口填写 8088，保存监听：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxGcbCZdJOsYWXkcftSU5UxNYVPxMur4yhagREADPSjo1oTGicUXfok2w/640?wx_fmt=png)

监听器配置完毕后，随便生成一个木马使机器上线，上线后执行命令，证明 CDN 配置生效：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7Oxgs2q5mrAnibEiaTCUZ3GCoxHrLYic3K7KC8JI5PSnpbxicCZsktEqqjDhw/640?wx_fmt=png)

执行命令回显：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxuSjnxxExPPJQ4HiculOLoe8x8aaUyxOgFfsMGvDITdUI9cpOBCibibHMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxXjvV4QBNmRwpQT4p5g9LYG2cPeiajE4zJZibbhIWRytfyUTkFSonjb0Q/640?wx_fmt=png)

在受害者主机上使用 wireshark 抓包分析比对，证明连接建立成功：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7Oxy2kjBiaXkbAeQW2qAMgpbcC4OlPCbMFJayia0HkS5N4SP4TTFpTReARQ/640?wx_fmt=png)在某云控制台中的 “CDN 监控” 功能模块中也能看到我在 4 月 14 日晚 18 时测试时请求 CDN 的次数统计，证明了配置的有效性：  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxCL0shDKV9K8nMxRAHALRVFWxPXHxlZCztc2ibylfc9fxoaCibNSXNsIQ/640?wx_fmt=png)

步骤三：https 证书的申请与配置

在某云的控制台中可以导入我们自己的 https 证书，之后配置生效，完整的 “域前置技术隐藏 C2” 就配置好了。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7OxT6iavl67ib53arJiaIW4Xfk7AuXb54IoTs1ia2o3syc36yx5gfc0hNUyicg/640?wx_fmt=png)

### 三、总结  

域前置技术隐藏 C2 的技术手段在目前的攻防对抗中是比较难以追踪溯源的，建议蓝队兄弟们在监控过程中最好不要过度依赖各种威胁情报对于 IP / 域名的白名单判定，多维度分析具体的安全事件才是根本。如有错误，可以留言一起研究，欢迎加我 wx：g3iicc4j2i2 进行技术交流，祝大家一切顺利~

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBSh0hPB3Ur6qszdWxQSP7Ox4l9TohSLicte40oOia2vM3o0kbPdFE68DCq1BFoexP5dKHIeQuG9zUqg/640?wx_fmt=png)

### 四、参考链接  

1、 https://www.cobaltstrike.com/help-malleable-c2  
2、 https://paper.seebug.org/1190/#_5  
3、 https://github.com/rsmudge/Malleable-C2-Profiles/blob/master/normal/amazon.profile  

end

  

招新小广告

ChaMd5 Venom 招收大佬入圈

新成立组 IOT + 工控 + 样本分析 长期招新  

欢迎联系 admin@chamd5.org

  
  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBR8nk7RR7HefBINILy4PClwoEMzGCJovye9KIsEjCKwxlqcSFsGJSv3OtYIjmKpXzVyfzlqSicWwxQ/640?wx_fmt=jpeg)