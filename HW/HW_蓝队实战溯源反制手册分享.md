\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/5EaSKW2G0eBLi6o9dEHdVQ)

前两天登录了一下防守方报告提交平台，看了一下提交报告模版并整理给下面各子公司方便整理上报 (毕竟只能上报 50 个事件，还要整合筛选)，发现比去年最大的区别就是追踪溯源类提交及分数的变化。

| 

描述



 | 

  

完整还原攻击链条，溯源到黑客的虚拟身份、真实身份，溯源到攻击队员，反控攻击方主机根据程度阶梯给分。

  




 |
| 

加分规则



 | 

  

描述详细 / 思路清晰，提交确凿证据报告，根据溯源攻击者虚拟身份、真实身份的程度，500-3000 分，反控攻击方主机，再增加 500 分 / 次。

  




 |

  

粗略的总结了一下规则里所谓的描述详细 / 思路清晰、虚拟身份、真实身份等细节的一个模版，并举例给大家参考。

  

**溯源结果如下：**

姓名 / ID：

攻击 IP：

地理位置：

QQ:

IP 地址所属公司：

IP 地址关联域名：

邮箱：

手机号：

微信 / 微博 / src/id 证明：

人物照片：

跳板机（可选）：

关联攻击事件：

（ps：以上为最理想结果情况，溯源到名字公司加分最高）

  

**我们拿到的数据：**

web 攻击事件 - 11  
攻击时间: 2020-08-17 09:09:99  
攻击 IP : 49.70.0.xxx  
预警平台：天眼 / 绿盟 / ibm / 长亭 waf

攻击类型: 植入后门文件  
处置方式: 封禁需溯源  
目标域名: 10.0.0.1  
www.baidu.com

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNuO59QwcB1lsIiaqtjNYVz0SbuzoyVd7tzXgPLW4hOyf7kCdic8aQia4ibQ/640?wx_fmt=png)

**流 · 程**

  

**1、针对 IP 通过开源情报 + 开放端口分析查询**

可利用网站：

https://x.threatbook.cn/（主要）

https://ti.qianxin.com/

https://ti.360.cn/

https://www.venuseye.com.cn/

https://community.riskiq.com/

  

主要关注点

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNgxh2v3dlkic2Jzias3f0Y02Vruz9uKfMzpFmzwIeCehIyyadWBnHKvvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNEbdP874HfkSmeUfRVphYdWkAgudcXYFRk9bhf5a8oOQ3TTibBe0fUtw/640?wx_fmt=png)

域名：可针对其进行 whois 反查

查询备案信息：http://whoissoft.com/

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNyNVVu9pGAm0Jia6FBes3AjTZvK5fTFg8ib3NMFFwCPn0cv3KHETpntiaw/640?wx_fmt=png)

  

端口：可查看开放服务进行进一步利用

可考虑使用 masscan 快速查看开放端口：

```
masscan -p 1-65535 ip --rate=500

```

  

再通过 nmap 对开放端口进行识别

```
nmap -p 3389,3306,6378 -Pn IP

```

  

端口对应漏洞：

https://blog.csdn.net/nex1less/article/details/107716599

  

**2、查询定位**

通过蜜罐等设备获取真实 IP，对 IP 进行定位，可定位具体位置。

定位 IP 网站：

https://www.opengps.cn/Data/IP/ipplus.aspx

  

**3、得到常用 ID 信息收集：**

(1) 百度信息收集：“id” （双引号为英文）

(2) 谷歌信息收集

(3) src 信息收集（各大 src 排行榜，如果有名次交给我套路）

(4) 微博搜索（如果发现有微博记录，可使用 tg 查询 weibo 泄露数据）

(5) 微信 ID 收集：微信进行 ID 搜索（直接发钉钉群一起查）

**(6) 如果获得手机号（可直接搜索支付宝、社交账户等）**

**注意：获取手机号如果自己查到的信息不多，直接上报钉钉群（利用共享渠道对其进行二次社工）**

(7) 豆瓣 / 贴吧 / 知乎 / 脉脉 你能知道的所有社交平台，进行信息收集  

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNs6OP4Shg2IdwTEiajgyn0WRZ5w4Pkiau7mgUicvwvvH8waaL5XeYpzRoA/640?wx_fmt=png)

  

**4、预警设备信息取证：**

上方数据一无所获，可考虑对其发起攻击的行为进行筛查，尝试判断其是否有指纹特征。

如上传 webshell : 

http://www.xxx.com/upload/puppy.jsp

可针对：puppy 昵称进行信息收集。

  

**5、跳板机信息收集（触发）：**

进入红队跳板机查询相关信息

如果主机桌面没有敏感信息，可针对下列文件进行信息收集

  

last：查看登录成功日志

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNcJiarLlGL3dulaBIfnvduibe9ymKfCZCcuVMAszPicSFf6Ge6SpzM7ECg/640?wx_fmt=png)

  

cat ~/.bash\_history  ：查看操作指令

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNIibZXibJbLMyfSmu8px0ByarHqU4KgZLyK61Hp4PUSLX2IzNwunDPdeQ/640?wx_fmt=png)

  

ps -aux  #查看进程

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNAgomwhmPallKRBgduynlG9sOxp4JPFTRyHoxcWMF6icjR6JQgJgTIWg/640?wx_fmt=png)

  

cat /etc/passwd

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPN5N2OSQmqX7haBNYhlL7UawDgDfDBPo6QMoVJtJHCV1qJrOFXDGzVPQ/640?wx_fmt=png)

  

查看是否有类似 ID 的用户

重点关注 uid 为 500 以上的登录用户

nologin 为不可登录

  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsia6b1lbLGMkxXcbcgVkeGPNle7xUPqvCjKkmQSDVubeY9z1nicsMFc7L7gLNgWgmLam03kiaNkZVKNQ/640?wx_fmt=png)

  

注意：手机号、昵称 ID 均为重点数据，如查不到太多信息，直接上报指挥部。
