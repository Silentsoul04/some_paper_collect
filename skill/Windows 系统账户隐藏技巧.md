> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xtBONAB3ZqlUir8c2UKauA)

  
![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

**原创稿件征集**

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬

> 本文转自 “乌云安全” 公众号，作者： Luckysec

### **0x001 系统账户隐藏**

渗透一台主机后，一般都想办法给自己留后门, 其中使用最多的就是账户隐藏技术。账户隐藏技术可谓是最隐蔽的后门，一般用户很难发现系统中隐藏账户的存在，因此危害性很大，本文就对隐藏账户这种黑客常用的技术进行复现。

### **0x002 新建特殊账户**

通过 net user 命令创建隐藏账号，并将其加入 administrators 组

```
net user test$ 123456 /add<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;">net localgroup administrators test$ /add
```

> 注：创建的用户名必须以 $ 符号结尾

添加后，该帐户可在一定条件下隐藏，输入 net user 无法获取信息

但是，在登录界面以及本地用户和组中却能够发现该帐户

### **0x003 赋予注册表权限**

使用 WIN+R 键，输入 regedit 打开注册表，展开注册表 [HKEY_LOCAL_MACHINE\SAM|SAM]

默认情况下 SAM 这个项里没有任何内容，这是因为用户对它没有权限。

在这个项的右键菜单里，为 administrator 用户赋予完全控制权限。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv79H2zWaGR639Mqz9icNfYBLsXEA3Ihqk0htTNfEVf8haaQT9LA23sVLQ/640?wx_fmt=png)

然后重新启动注册表，即可看到如下效果

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7cCJJ0nWPNMngwt7YCChLfLA1Wub20ru8ZYrjiaX8ju1Ekrryl8MT3ww/640?wx_fmt=png)

**0x004 导出注册表**  

在 [SAM\Domains\Account\Users\Names] 项里显示了当前系统存在的所有账户，选中 test$ ，在其右侧有一个（默认）选项，类型为 0x3eb 的键值。

其中的 3eb 就是 test$ 账户 SID 的结尾，即 RID（这里使用十六进制表示）。另外，在 [SAM\Domains\Account\Users] 里还有一个以 3eb 结尾的子项。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7ypSgVNVgaSTAklA28h5jnFia1urwjJnicVOL62qaNXiaEKiaFZxjcmjaHg/640?wx_fmt=png)

这两个项里都是存放了用户 test$ 的信息。在这两个项上单击右键，执行 导出 命令，将这两个项的值分别导出成扩展名为 .reg 的注册表文件。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7P4DKWFscvnSROY2YbGrOfeOA0qUDRAAcBGFiamQ7mT7mzOTo1ARichhQ/640?wx_fmt=png)

**0x005 删除特殊账户**  

然后将 test$ 账户删除

```
<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;">net user test$ /del
```

再次重启注册表，此时上述两个项都没了

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7F9JO7LeCxTfFsOPdwOoo1PZtG6xpMNc6JrUUmoZ2wgrdZdGvEPD1zA/640?wx_fmt=png)

**0x006 导入 reg 文件**  

下面再通过命令行将刚才导出的两个注册表 .reg 文件进行重新导入

```
<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;">regedit /s test.reg<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;">regedit /s test1.reg
```

此时在注册表里就有了 test$ 账户信息

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7qYiapngcFibY0O00Rjibia9W6KsjdvuGNGt7nQoibdia9NF2TSxMaG0DUIww/640?wx_fmt=png)

隐藏账户制做完成，控制面板不存在帐户 test$  

通过 net user 无法列出该帐户

通关 WIN+R 键，输入 lusrmgr.msc 打开本地用户和组，查看用户，也无法列出该帐户

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7ibtOVE3CHfFBndj94VFrLRia12sTpHFFm9aThjtia2M7dcbdNNPADKGMw/640?wx_fmt=png)

但可通过如下方式查看，这种方式的前提是必须已经清楚隐藏账户的名字，所以一般是管理员是不会发现的

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7cKcDcgaxT66Oz8iay92g1W3icNHUPOXZRwBfDkXRL9nWO6VshmQWO7sQ/640?wx_fmt=png)

使用这个隐藏账户可以登录系统，但缺点是仍然会产生用户配置文件，下面再对这个账户做进一步处理，以使之完全隐藏。  

### **0x007 隐藏用户配置文件**

展开 [SAM\Domains\Account\Users\Users] 注册表项中，找到 administrator 用户的 RID 值 1f4，展开对应的 000001F4 项，其右侧有一个名为 F 的键值，这个键值就存放了用户的 SID 。

下面将这个键值的数据全部复制，并粘贴到 000003EB 项的 F 键值中，也就是将 administrator 用户的 SID 赋给了 test$

这样在操作系统内部，实际上就把 test$ 当做是 administrator，test$ 成了 administrator 的影子账户，与其使用同一个用户配置文件，test$ 也就彻底被隐藏了。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7zgcOne3bOOMPIiblQbs62M5DXxViaKSG4nh0yKgQT49khoibdzEfmZjcA/640?wx_fmt=png)

**0x008 删除隐藏用户**  

使用普通的账户删除命令是无法删除隐藏账户的，提示用户不属于此组

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7TSAFDa6VKRHpWmMttW8mSOXA8mOXc3GDZ1RVJZWicK0maP7uK0q40Ew/640?wx_fmt=png)

只能将删除注册表 [HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users] 下对应帐户的键值 (共有两处)，也就是刚刚创建 test$ 所产生的那两个注册表文件  

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavxPFxWraLfMAhj1TsQRwVv7qYiapngcFibY0O00Rjibia9W6KsjdvuGNGt7nQoibdia9NF2TSxMaG0DUIww/640?wx_fmt=png)

删除完这两个注册表项后，即可完全清除隐藏账户 test$  

### **0x009 防御隐藏用户**

打开注册表的 [HKEY_LOCAL_MACHINE] 项，检查该项下的 [SAM\SAM\Domains\Account\Users] 是否有可疑账户

默认管理员权限无法查看注册表，需要分配权限或是提升至 Sytem 权限隐藏帐户的登录记录，可通过查看日志获取

### **参考文章**

https://www.cnblogs.com/wjvzbr/p/7906855.html

**实操推荐：Metasploit 后渗透入门  
**  

https://www.hetianlab.com/expc.do?ec=ECIDee9320adea6e062018020615520500001&pk_campaign=weixin-wemedia#stu  

复制链接体验至 PC 端体验吧~  
学习拿到 meterpreter 后的渗透学习。这一阶段也被称为后渗透阶段。  

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LfbzQb75ZqoK2T2YO9XTQYD0aDUibvcxdbLRqzCwlkYcn0HppvXpZuenRzjX8ibhzcibJJge9Bw9xc8A/640?wx_fmt=gif)

  

**戳**

**“阅读原文”**

**体验免费靶场！**