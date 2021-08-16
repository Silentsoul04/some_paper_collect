> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EgvbSpK5Gn3u1N1Q0G9rdw)

_**01**_

前言

此次渗透为一次授权渗透测试，门户网站找出了一堆不痛不痒的小漏洞，由于门户网站敏感特征太多而且也没什么特殊性就没有截图了，后台则是根据经验找出并非爆破目录，涉及太多敏感信息也省略了，我们就从后台开始。

_**02**_

正文

拿到网站先信息收集了一波，使用的宝塔，没有 pma 漏洞，其它方面也没有什么漏洞，没有捷径走还是老老实实的渗透网站吧。  

打开网页发现就是登录，果断爆破一波，掏出我陈年老字典都没爆破出来，最终放弃了爆破。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribSc6OK99g5P38Qa4r7I4kqByL9dlVH6AAOKB9y1e4LNFjzcOeZ36kyhaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScRSp3SJp0vtQt4Gw7VFso9cg7dSWammL2iaXmhCt2mEcBCJkZ1J0RFJg/640?wx_fmt=png)

跑目录也没跑出个什么东西，空空如也，卡在登录这里找回密码这些功能也都没有，CMS 指纹也没查到，都快要放弃了，瞎输了个 login，报了个错（准确说是调试信息），提起 12 分精神查看。

小结：渗透这东西还是要讲究缘分的。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScBzJhfCYYGMiaWL1ictleZUYScmdbAibiaPibZUtvFuXxPJJK54Iqibf1SBTQ/640?wx_fmt=png)

往下继续看

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScgiaykFB1f1A2UDAHrTrsLIOdYgoZall0DKGPPZMbYxCmH9dibyNW1cWA/640?wx_fmt=png)

Reids 账号密码都有，上面还有个 mysql 账号密码，但是端口都未对外网开放，只能放弃了

但下面 ALIYUN_ACCESSKEYID 跟 ALIYUN_ACCESSKEYSECRET 就很关键。

利用方式可以手动一步一步来，但是已经有大神写出了工具，不想看手工的直接滑到最后部分。

说句废话：手工有手工的乐趣，一步一步的操作会让你做完后有种成就感，我个人觉得手工其实是种享受，工具呢只是为了方便，在红蓝对抗中争分夺秒时使用。

### **手工篇**

首先用行云管家导入云主机，网站地址：https://yun.cloudbility.com/

步骤：选择阿里云主机 -> 导入 key id 跟 key secret -> 选择主机 -> 导入即可（名字随便输）

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScqcl4s2Y64cliaEPCR1767cdV4T9sWtupSibrrJpghJibFLhkYQ95tkF0A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScIRQcFArqmfIRyrs8xvCM4YWtJms9TAjfuk4iaqTLibYwFsicKFwticVPxQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScaJfUqE4z2OoYdTk8B1tf9OuUiceT1rXzM2iclVCTf6wyMJhcR2IcbgoA/640?wx_fmt=png)

导入成功后在主机管理看得到

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScIc5qm7dr1IQdXFPA5Edl40OxxTab2NUBnMhXDV0bYhdk2qjaVnbwTg/640?wx_fmt=png)

点进来查看详情，这里可以重置操作系统密码，但作为渗透，千万千万千万不要点这个，不能做不可逆操作。我们用这个只是为了得到两个数据，就是实例 ID 以及所属网络，拿到就可以走人了。往下看

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScHIY3J7iaB8uzUgIkqvibrSCQVu9lXK2qPyWzSDbwWFDwp8aMm4PVHJsA/640?wx_fmt=png)

这里我们打开阿里 API 管理器，这个是阿里提供给运维开发人员使用的一个工具，

https://api.aliyun.com/#/?product=Ecs

点击左边的搜素框输入 command，我们会用到 CreateCommand 跟 InvokeCommand，CreateCommand 是创建命令，InvokeCommand 是调用命令。继续往下看  

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribSc1yXehrLAgMEv9p7DyEFfuSru0zW7YlA50JjAtIEibWVHYEGo0qtJVnQ/640?wx_fmt=png)

Name 部分随意

Type 指的是执行脚本类型

    RunBatScript：创建一个在 Windows 实例中运行的 Bat 脚本。

    RunPowerShellScript：创建一个在 Windows 实例中运行的 PowerShell 脚本。

    RunShellScript：创建一个在 Linux 实例中运行的 Shell 脚本。

CommandContent 为执行命令，需要注意的是这里是填写 base64 编码。

填写完后选择 python

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScFLYCmT0CO5KSxJSZNa8LJATJc0jC4cRkPHjuCoUr4pDUib44HRQUDUA/640?wx_fmt=png)

点击**调试 SDK 示例代码**，此时会弹出 Cloud shell 窗口，并创建一个 CreateCommand.py 文件，用 vim 编辑器打开 CreateCommand.py，修改 accessKeyId 与 accessSecret。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScqAeuZ8UumNuZqwSyLhMkWBibHpQbEINQKYY7yHqNYOMFyqrlnmTgu9Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScoM4GVJbBWJJfaJTNgHFNuEqZSgkrdicOwr5HRxBkwo2ffNNXM2LLKoQ/640?wx_fmt=png)

执行 CreateCommand.py，会返回一个 RequestId 与 CommandId，记录 CommandId，后面调用命令会用到。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScBE7czOcxicKZuXHIpRicVgicRAibTmDK5oGhjZTXQtqibV3iatJjDGWPD2mQ/640?wx_fmt=png)

打开 InvokeCommand

RegionId 填写行云管家中的所属网络

CommandId 填写刚刚执行 CreateCommand.py 返回的 CommandId

InstanceId 填写示例 ID，行云管家中获取到的那个

继续点击**调试 SDK 代码**，会生成一个 InvokeCommand.py 文件，同样用 vim 编辑器把 accessKeyId 与 accessSecret 修改了。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScfCqYTQiaFqlLrnG8ntYaecWnZZwBJqDLHeC7cBHopSxFdhibqObyVhdw/640?wx_fmt=png)

修改完成后使用 nc 监听端口，执行 InvokeCommand.py。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScWV8vIzqkd7OGnQH4xszFoHurKCiagO5vGxQLMeq2jBA6KMeaFBWseJg/640?wx_fmt=png)

成功执行命令反弹 shell，收工。

### **工具篇**

**使用方法：**

查看所有实例信息

```
AliCloud-Tools.exe -a <AccessKey> -s <SecretKey> ecs –list
```

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribScjTQQm06yTtUBbzhQ0moFw6OgNAdEjkdxFBicAK8ibibzpk29ibZJu2NFsg/640?wx_fmt=png)

当拿到示例 ID 后就可以执行命令了

执行命令

```
AliCloud-Tools.exe -a <AccessKey> -s <SecretKey> ecs exec -I <实例ID> -c "执行命令"
```

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxI5hGqt5EdOUibOTj95ribSciaW80icLibQxyicTNkvmkrZf5ZCXl8ichm62icGoC2Fgq1t6U2qiaHffXcrRA/640?wx_fmt=png)

_**03**_

总结  

Access Key 一般会出现在报错信息、调试信息中，具体场景没法说什么情况。本次遇到也是意外收获，因此我在网上也查阅了很多文章，并没发现什么技巧。如果是 APP 在 APK 中存放 Access Key Web 页面 / JS 文件等 Github 查找目标关键字发现 Access Key 与 AccessKey Secret 拥有 WebShell 低权限的情况下搜集阿里云 Access Key 利用。  

免责申明：    

    本项目仅进行信息搜集，漏洞探测工作，无漏洞利用、攻击性行为，发文初衷为仅为方便安全人员对授权项目完成测试工作和学习交流使用。       请使用者遵守当地相关法律，勿用于非授权测试，勿用于非授权测试，勿用于非授权测试~~（重要的事情说三遍）~~，如作他用所承受的法律责任一概与**凌晨安全**无关！！！

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicyHabdCJq88b5IfbicnbDsUbahfHB0mhhQZTqZRrnpWvlZHnatcb06ibS7eebEe8lCKOjs8mUOK6GIg/640?wx_fmt=png)