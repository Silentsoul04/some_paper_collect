> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GQJhysIMc0sgu1BLP5QcYg)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV031HpDUGvxgaF1ccTyES3hlFhnnz1EyjpVqLaZpRsmuokD9UxkvMmZ1OANaU7eWia5qX4wWgFzSyA/640?wx_fmt=png)

PickleC2 是一个用 python3 编写的简单 C2 框架，用于帮助渗透测试人员的社区参与红队活动。

PickleC2 能够为后期开发和横向移动导入您自己的 PowerShell 模块或自动化该过程。

特征
--

测试版有一个植入物，它是 powershell。

1.  PickleC2 是完全加密的通信，即使在通过 HTTP 通信时也能保护 C2 流量的机密性和完整性
    
2.  PickleC2 可以毫无问题地处理多个侦听器和植入程序
    
3.  PickleC2 支持任何想要添加自己的 PowerShell 模块的人
    

在即将到来的更新中，pickle 将支持：

1.  去植入
    
2.  不使用 System.Management.Automation.dll 的 Powershell-Less Implant。
    
3.  将支持可锻 C2 配置文件。
    
4.  将支持 HTTPS 通信。注意：即使是 HTTP 通信也是完全加密的。
    

安装
==

PickleC2 是一个开源的，可以在 Github 上找到。PickleC2 目前只支持 linux，你可以通过 https://github.com/xRET2pwn/PickleC2 下载

```
git clone https://github.com/xRET2pwn/PickleC2.git
cd PickleC2
sudo apt install python3 python3-pip
python3 -m pip install -r requirements.txt
./run.py
```