> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/t3bG_SgnKvqFqFrUkh8GcQ)

[![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smjictCEibz7FOLcXD4geBCcOWe8OILoRNOmxBAibnvnnq8FBmCuvYHc5waw/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDQ2NjExOQ==&mid=2247494461&idx=1&sn=2c4d86a41041df8970eb58b8d6e4be4f&chksm=ec1ddb15db6a5203313fdda2545255d801111c1a10475bdedb425b0017417c63cad6b1ab0a07&scene=21#wechat_redirect)

对于 burp 和 mitmproxy 工具而言， 通常用于拦截浏览器的 http 流量，对于一些命令行工具，比如 wget、curl 或者 python 编写的脚本，无法直接使用的 burp 截取数据，很少有文章提到这方面的应用，本文就来测试一下各种命令行工具如何使用 burp 抓取数据。  

通常来说，使用 burp 截取数据，需要两步：

1、让命令行工具代理流量到 burp

2、让命令行工具信任 burp 的证书（CA）或者忽略信任

### 案例一 代理 curl 和 wget

curl 和 wget 是 linux 下默认的 web 页面访问工具

#### 1、让 curl 和 wget 的流量通过 burp 代理

需要设置全局变量，将本地默认代理设置为 burp 的代理服务地址和端口，可以使用如下命令：

```
export http_proxy=localhost:8080
export https_proxy=localhost:8080
curl ifconfig.io
wget -O /dev/null ifconfig.io
```

也可以连在一起使用：  

```
http_proxy=localhost:8080 https_proxy=localhost:8080 curl ifconfig.io
http_proxy=localhost:8080 https_proxy=localhost:8080 wget -O /dev/null ifconfig.io
```

最终结果如图：  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smjKOqp5DLsHzsLNITWAjSnSOl5gtO4NwRdjapccIz7L60TSWN8z8SLdg/640?wx_fmt=png)

#### 2、让 curl 和 wget 信任 burp 的 CA

如果不信任的话，在使用 curl 和 wget 访问 https 网站时报错，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smj2nI2zQKa0XJjvQZGJaH5cXs8t7Kq0kttlSryL3icnQPFibfH3VTEPZAQ/640?wx_fmt=png)

对于这种情况，有两种解决办法：

**1、禁用证书信任验证**

curl 可以使用 `-k` 参数，wget 可以使用 `--no-check-certificate`

**2、配置系统信任 Burp 的 CA**

我们需要访问 https://localhost:8080 下载 Burp 的证书，然后执行下面的操作：

```
mkdir ~/certs
wget -O ~/certs/burpca.der http://localhost:8080/cert
cd ~/certs
openssl x509 -inform DER -in burpca.der -out burpca.crt
```

如果使用的是 `mimtproxy` 证书需要放在目录 `~/.mitmproxy` 下  

在 Moc OSK 系统，只需要双击下载的 der 文件，让它受机器上所有用户的信任，导入后，搜索 "PortSwigger" 并打开证书，在 SSL 选项中选择 `Always Trust`

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smj8fZ1Ypnl53HficFGEy7ITR72ZYofYh6lDLpd8BuLFkcR8os19TANHdQ/640?wx_fmt=png)

在 windows 系统，同样双击 der 文件，选择安装证书，选择受信任的根认证机构来安装和信任 Burp 的 CA：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smjOvFQNiaoTHALE7tWXjnWpP0YKibAEb1yzicm6aF1IXFAWIXDlptNbD2JQ/640?wx_fmt=png)

在 Linux 下，信任证书的存储位置在 `/usr/share/ca-certificates`，复制 burpca.crt 到该目录下，然后运行：

```
sudo update-ca-certificates
```

然后不需要使用跳过验证的参数就可以用 burp 截取到 wget 和 curl 的流量了：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smj64FJy25Bjbwc3y4xOx0JSJrdfdZPrEiaNLHRmLL24xzCSZBzsx1m6Zg/640?wx_fmt=png)

### 案例二 代理 Java 的 jar 程序

通常 jar 程序是没有设置代理的选项的，但是如果遇到这样的情况该怎么办？

在启动 jar 程序时可以使用 JVM 的一些属性来使用代理，如：

> *   `http.proxyHost`
>     
> *   `http.proxyPort`
>     
> *   `https.proxyHost`
>     
> *   `https.proxyPort`
>     
> *   `http.nonProxyHosts`
>     

随便找一个 jar 的程序来测试，比如使用 `acli` 获取云实例上的 jira 版本

```
java -jar acli-9.1.0.jar -s https://greenshot.atlassian.net -a getServerInfo
Jira version: 1001.0.0-SNAPSHOT, build: 100119, time: 2/6/20, 6:26 AM, description: Greenshot JIRA, url: https://greenshot.atlassian.net
```

可以在指定 jar 之前，将代理参数添加进去，如下：  

```
java -Dhttp.nonProxyHosts= -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=8080 -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=8080 -jar acli-9.1.0.jar -s https://greenshot.atlassian.net -a getServerInfo
```

执行之后收到一个 SSL 的错误：  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smj9hgWATH4Q2aQEXBuyCd99rY9ZvFLv3csdibP9EPgNsCuibWIU97XnDNA/640?wx_fmt=png)

Java 实际上是使用自己到密钥存储，所以还需要添加证书。

默认的证书存储位置在 `$JAVA_HOME/lib/security/cacerts`，如果不知道 `$JAVA_HOME` 的位置，可以使用下面的命令查询：

```
java -XshowSettings:properties -version 2>&1 > /dev/null | grep 'java.home'
    java.home = /Users/RonnieFlathers/.sdkman/candidates/java/11.0.3-zulu
```

可以使用 Java 的 keytool 程序来为 Java 的密钥存储添加证书，该程序所在位置 `$JAVA_HOME/bin/keytool`，导入命令如下：  

```
$JAVA_HOME/bin/keytool -import -alias burpsuite -keystore $JAVA_HOME/lib/security/cacerts -file $HOME/certs/burpca.crt -trustcacerts
```

提示的密钥存储密码，默认是 `changeit`，然后是否信任证书选择 `yes`：  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smjjcy2ylAGcuVcWm1MuDshh4HZibibLNOs1bBJVbuvEJF63MuIbcpO9h9w/640?wx_fmt=png)

现在，在来执行上面的命令，可以看到数据：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdvuhDianSK18U8QvWTC1smjiadia992eLQP1nHWJtmX67mTlDib3NhYodEwVamLl5ZESSA1wGOP1U1oQ/640?wx_fmt=png)

### 总结

以上方法对于测试一些二进制文件的数据请求方式有很大的帮助，能够了解其对外发送数据包的情况，来猜测二进制文件的执行原理，欢迎试用。

![](https://mmbiz.qpic.cn/mmbiz_gif/sGfPWsuKAfdvuhDianSK18U8QvWTC1smjLnicbphtypgvecNktJQquOiaiaeCFTZgE8vOZT9WAibHeHhExHLia2fAbFA/640?wx_fmt=gif)