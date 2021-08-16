> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/AlD3nWvhns_UeS43laT3Zw)

  
![](https://mmbiz.qpic.cn/mmbiz_png/FIBZec7ucCgvIMaxEuUH2UlN1dB1eRJibSrenep4vU6AgLfmicMSXgY4YwWx36voAicBE9fQP7FMELsUz3snNxicWQ/640?wx_fmt=png)

**利用 Account Operators 组实现接管域控或权限提升**

****目录****

        利用基于资源的约束性委派进行权限提升

        Write Dcsync Acl dump 域内哈希接管域控

* * *

     在域渗透的过程中，我们往往只会关注 Domain admins 组和 Enterprise Admins 组，而会忽略了其它组。今天，我们要讲的是 Account Operators 组。该组是内置的本地域组。该组的成员可以创建和管理该域中的用户和组并为其设置权限，也可以在本地登录域控制器。但是，不能更改属于 Administrators 或 Domain Admins 组的账号，也不能更改这些组。在默认情况下，该组中没有成员。也就是说，该组默认是域内管理用户和组的特殊权限组。可以创建和管理该域中的用户和组并为其设置权限，也可以在本地登录

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUI4JMjnUSmHnPfpicZUib4EpPUycD4SpL0A0jjUhew5SEdeTg9zfKibZBA/640)  

    在实际环境中，某些企业会有专门的管理用户账号的账号，这些账号就会分配在 Account Operators 组中。在渗透过程中，如果我们发现已经获得权限的用户在该组中的话，我们可以利用其特殊权限进行 dump 域内哈希或本地权限提升。

  

利用基于资源的约束性委派进行权限提升

域内如果域内没有安装 Exchange 服务器的话，我们可以利用基于资源的约束性委派攻击除域控外的域内其他所有机器，获取这些机器的本地最高权限。

查看 Account Operators 组内用户，发现 hack 用户在内。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUdVviaf1ZyrogvMm4N7zw6CA58EL5fbKXqgic3CtJficpzPGVMWpIxic1FQ/640)

    我们获取到了某台机器的权限，当前登录用户为 hack，但是 hack 并不在本地机器的管理员组中。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUjhRKHraEM3NNF72vV5Fcez7Unm0lyStXBnMicVSg3H0U5icN9UAjYzgQ/640)

于是我们可以新建机器账号，然后配置机器账号到指定主机的基于资源的约束性委派

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUpgNqn8sLibroicLUEzkDN9gUamAtFBTtZQCJfYOO5iaMiaoc17O7XRUvNA/640)  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUKMl7cV92CmedAW57BNQzQpw2fS2t1WiaGjDicvCwUdVFNgelFyhwaIJA/640)               最后利用新建的机器账号模拟 administrator 用户访问指定主机的 cifs 协议生成票据，导入票据后，就获取了目标机器的最高权限。

```
python3 getST.py -dc-ip 10.211.55.4 xie.com/test:root -spn cifs/win7.xie.com -impersonate administrator
export KRB5CCNAME=administrator.ccache
python3 psexec.py -k -no-pass win7.xie.com
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUA3hya5Y5DzJVYGcgaCv9eQJfoVAmU2PgicRntibljHcDDSg3kfl1Yb8g/640)

  

Write Dcsync Acl dump 域内哈希接管域控

    如果域内安装了 Exchange 服务器的话，我们可以将指定用户添加到 Exchange Trusted Subsystem 组中，由于 Exchange Trusted Subsystem 用户组又隶属于 Exchange Windows Permissions。Exchange Windows Permissions 这个组默认对域有 WriteACL 权限。因此我们可以尝试使用 WriteACL 赋予指定用户 Dcsync 的权限。  

查看 Account Operators 组内用户，发现 hack 用户在内。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUdVviaf1ZyrogvMm4N7zw6CA58EL5fbKXqgic3CtJficpzPGVMWpIxic1FQ/640)

    我们获取到了某台机器的权限，当前登录用户为 hack，但是 hack 并不在本地机器的管理员组中。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUjhRKHraEM3NNF72vV5Fcez7Unm0lyStXBnMicVSg3H0U5icN9UAjYzgQ/640)

将 hack 用户自身加入到 Exchange Trusted Subsystem 组中

```
net group "Exchange Trusted Subsystem" hack /add /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQU7P5UPses0qfJMoQjLxBCAh1ykPvdbF8O3KoydOqhxibI6Y13sUHl3hw/640)

注意，这里需要将 hack 用户先在当前机器注销一下，重新登录。或者登录其他机器。

然后赋予 hack 用户自身 dcsync 权限

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUO2Mln5D2GxxhDy3eRbGPiaveodZ8MRQKgtmYfGx3QibsC7zDjBxjJKkA/640)

使用 hack 用户 dump 域内任意用户哈希，即可接管整个域。

```
python3 secretsdump.py xie.com/hack:P@ss123@10.211.55.4 -dc-ip 10.211.55.4 -just-dc-user administrator
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUG0ovnR4rUQNcfXql2faHpLcXT7smdfNvTiblNnREwHcl7Yph1aibRD6A/640)

@深蓝攻防实验室

如果想跟我一起讨论，那快加入我的知识星球吧！https://t.zsxq.com/7MnIAM7

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUJ2ed5fxYvws9QrsiaaXtMqRxaiaWFryhXYVpiaDxVUPA2vBQvj0G0uKicQ/640)