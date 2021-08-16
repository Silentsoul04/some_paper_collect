> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Sep-0DFHopM6SyG5SIFuVg)



**一、Charles 原理**

• 对 Http 进行抓包，对手机设置代理，转发

• 对 Https 进行抓包，使用的原理就是中间人技术（man-in-the-middle），

Charles 会动态生成一个使用自己根证书签名的证书

Charles 接收 web 服务器的证书，而客户端浏览器 / 客户端 接收 Charles 生成的证书，以此客户端和 Charles 之间建立 Https 连接，Charles 和 Web 服务器之间建立 Https 连接，实现对 Https 传输信息的抓包。

如果 Charles 根证书不被信任则无法建立 Https 连接，需要添加 Charles 根证书为信任证书。

Charles 官网地址（有乱码需要自己改）：

https://www.charlesproxy.com/download/

**个人收藏版（稳定版 - 无乱码版本）**

**二、Charles 破解**

破解方法 1：通过替换 Charles.jar 破解

```
下载地址：https://www.zzzmode.com/mytools/charles/ 
1、输入RegisterName(此名称随意，用于显示 Registered to xxx)
2、选择本地已安装的版本，点击生成，并下载Charles.jar文件
3、Mac安装地址：替换本地 /Applications/Charles.app/Contents/Java 目录下的Charles.jar文件
 Windows安装地址：替换本地安装目录下的charles.jar文件
```

破解方法 2：通过验证 License Key 破解

```
Registered Name: https://zhile.io
License Key: 48891cf209c6d32bf4
```

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaazy91U5hTESbUVoicAicaicLlWxd0LLZuIvXOXxaq8mia3SgCWsG7UrBPiaA/640?wx_fmt=png)

重启 Charles，点击 Tools -> help，第二栏显示 Registered to xxx， 即破解成功！

（破解成功后的 charles 在启动时，左下角的倒计时也没有了）


Charles 破解成功

**三、设置抓取 https 协议**

**1、Proxy 端口设置**

选中 “Charles -> Proxy -> Proxy Settings”，设置 Port 为 8888，选中“Enable transparent HTTP proxying” 选项；

Charles 代理的端口  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaaIeJYjFxNyELibtRcuUZFUf7OH3HpFkH8fSuvuFURh9gkEGc5HWLdZvw/640?wx_fmt=png)

**2、在弹框中选择 “Grant Privileges” 信任；**

如果点击 “Not Yet” 或者关闭按钮，Charles 将会取消 Mac 代理，需要手动设置，设置时会再次弹出窗口；

**3、SSL 证书 - 安装与信任三种证书:**

钥匙串根证书, 模拟器证书, 手机和浏览器证书

选择安装 Help -> SSL Proxying -> 以下证书：

```
Install Charles Root Certificate；#钥匙串根证书，选择始终信任
Install Charles Root Certificate on a Mobile Devices Or Remote Browser #手机和浏览器证书，在弹框提示下安装手机ssl证书到手机
手机安装地址：chls.pro/ssl，直接在手机浏览器的地址栏输入，安装允许就好了，下载的文件需要改后缀.pem为.crt；
```

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaaqBCzTl2qiaBqeawmkqMf7EemetkicvwJ1ZNoUl6JCcbdjtZnYbkG1bcQ/640?wx_fmt=png)

打开钥匙串, Charles Proxy CA 证书, 一开始是不被信任的；选择始终信任;

**4、SSL 代理和端口号设置**

```
https请求被拦截：https请求会显示unkonwn 就是不能解析https请求；
（1）Proxy-> SSL Proxying Settings->SSL Proxying，选中"Enable SSL Proxying"；
（2）添加host，端口号为443。这里是把所有的host都设置进去；当然也可以设置指定的host，端口不变；
（3）请求就可以解析出来了
```

**四、手机连接 Charles 配置**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaaEmxv1eGibkJIScBduobia8ibfKFkAiaWSGMyZvePThSTibMo9gJ4Exq8QUA/640?wx_fmt=png)

```
注意：手机的无线网要和电脑的无线网保持一致,必须是同一wifi;
```

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaaADiccpsr9YQ60Fwfq4rOfHJ1hPeUT6ylIL2rRUJago47WW8S8T7ibjNw/640?wx_fmt=png)

**五、Charles 抓包工具断点修改返回内容**

在 structure 窗口，在要打断点的数据右键选择 “Breakpoints”


![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaa8FQ9cIByjgHO4ZDElKdibJZAcqNDJXI4MvJYPy9iabcpcicLIy019rG2g/640?wx_fmt=png)  


在对目标服务器进行访问，此时会跳转到 Breakpoints TAB 页面，

对目标服务器进行仿真，让 charles 抓取服务器返回内容信息，如下图：


![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEdzZh80dnoM0Iwmv9GVbaajic5t2WojqOySiaWcBCIictdjYN7gMpRpWwFicycXFfMpG1RWrWtuFfF7w/640?wx_fmt=png)  

在 Edit Response 中可以对服务器返回内容进行修改。点击”Execute” 进行提交，

超级方便。

