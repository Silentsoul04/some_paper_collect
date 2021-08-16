> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1n1ZhdY2uA9LcK2Tw7gXoQ)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**前言**_

目前项目当中，遇到了客户提供的加固 app 使用了某 tong 的加解密，以前常用的 xserver(https://github.com/monkeylord/XServer) 对于这种加解密无法 hook 到传入传出了，大佬最近 996 也没空给我改 bug 了，没办法用了，最后使用了 l 总的 HTTPDecrypt(https://github.com/lyxhh/lxhToolHTTPDecrypt/tree/master/HTTPDecrypt), 也学习了一下 httpdecrypt 的思路，最后自行编写 frida 脚本快速解决 app 中的加密问题。

现在 app 加密的越来越多了，正常情况下用 burp 抓 app 的包是这样的，通过对 httpdecrypt 和 xserver 思路的学习可以通过 frida 快速修改 body 中的密文了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVYicBZmiaQPXI0Gen63ZPfzh2KB9O7xWwOYtSVJBV05bZAHzIzdTU2FxwP3nQYY34JUles3VM3ob1w/640?wx_fmt=png)

_**No.2  
**_

_**思路**_

  

对于现在 app 中的加解密，基本都使用了非对称加密，因此自行构造加解密机制的方法耗时比较久，因此这里使用了 hook 加解密方法的传入传出来解决加解密，在这种思路下，解决 app 的加解密只需要三步：

1、分析出 app 中的加解密方法

2、hook 加解密方法

3、搭建 http server，将 hook 到的信息发送到 http server 上，方便 burp 抓包

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVYicBZmiaQPXI0Gen63ZPfzhWneibQbwO3paemOeLfRTlia3sR6KMumy4n8WpzMRlczf92tTTH8VEBag/640?wx_fmt=png)

_**No.3  
**_

_**分析应用加解密方法**_

  

一开始就遇上了某加密的壳，先上万能的 fdex2 脱壳看看代码，结果发现是用的类抽取的壳，具体怎么脱类抽取的壳，大佬不愿意 py 给我，但是好在类抽取脱出来也能看见具体的类名和方法名。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVYicBZmiaQPXI0Gen63ZPfzhiciaSFGleibN8Ol6JrOSpVtKefGAaKluicXbTgm8KDq5Yu2Bs2I4fL3rRQ/640?wx_fmt=png)

接着将脱下来的 dex 文件用 jadx 全局搜索一下 encrypt 关键字，还是发现了很明显的加解密方法的，不过由于类抽取的原因，只能发现方法的传入传出，这时候没办法了就只能用 l 总的 httpdecrypt 动态调试看看。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVYicBZmiaQPXI0Gen63ZPfzhwXnviawk1GSHWN3jbc4euicTcsARDTGyibH4ZujzdeIplVibP0m5ibjvibhA/640?wx_fmt=png)

具体的调试方法，httpdecrypt 的 github 上面有很详细的说明，我就不赘述了，通过对整个类的 hook 和方法的遍历，找到了 decryptDataWithSM 方法就是加密方法了，而且第二个传入的参数就为传入的明文，那我们 hook 这个参数，修改它的话，就可以先于加密前修改请求了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVYicBZmiaQPXI0Gen63ZPfzhkQIrbEcGHGhMG5v4oLLomKsC0xLNcibQnrbgRXQ0OomDrGrUTibJda9g/640?wx_fmt=png)

_**No.4  
**_

_**利用 frida hook 加密方法**_

  

现在几家做加固的手机厂商，基本都防 xposed hook 了，但是对 frida hook 都防的不咋地，而且垃圾电脑也带不动 android studio 了，所以用 firda 方便很多，hook 进加密的类后，我们考虑到要 hook 到请求的明文和返回包的明文，因此需要分别 hook 加密方法的传入和解密方法的传出，具体代码如下

```
Java.perform(function () {
    console.log("[*] Hooking ...");
  //利用frida hook到加密的类中
    var hclass = Java.use("com.*.security.CryptoUtil");
    // hook 加密方法
    hclass.encryptDataWithSM.implementation = function (a,b,c) {
      //hook到加密方法的第一个参数
        send(arguments[1])
      //获取处理后的参数
        var op = recv(function(value) {
            console.log("[*] js recv encryptdata content: " + value);
            b =  value;
        });
        op.wait();
      //将处理后的参数返回到应用的加密方法中继续加密
        return this.encryptDataWithSM(a,b,c);
    };

    // hook 解密方法
    hclass.decryptDataWithSM.implementation = function(a,b,c){
      //获取解密后的返回值
        var getVal = this.decryptDataWithSM(a,b,c)
       //将返回值发送到python脚本中进行处理
        send(getVal)
        var op = recv(function(value){
            console.log("[*] js recv decryptdata content: "+value);
            getVal = value;
        });
        op .wait();
        return getVal;

    };
});
```

这里介绍一下 frida 的 send 函数，通过使用 send 函数将 js 中的数据传入到 python 脚本中，由 python 脚本进行处理。这样我们就能在 python 脚本中获取到请求的明文和返回包的明文了。

_**No.5  
**_

_**将明文发送到 burp 中**_

既然都要改请求了，肯定还是要用 burp 才习惯吧，所以还是要把请求发送到 burp 中去改包，这里我选择了利用 flask 框架，在本地搭建了一个将接受到的请求直接返回的 http 服务。

```
from flask import request, Flask, jsonify
import json

app = Flask(__name__)
app.config['JSON_AS_ASCII'] = False


@app.route('/test', methods=['POST'])
def post_Data():
    payload = json.loads(request.data)
    return jsonify(payload), 201


if __name__ == '__main__':
    app.run(debug=False, host='0.0.0.0', port=8888)
```

然后在上面的 python 脚本中，将获取到的明文信息直接通过 requests 库走 burp 代理，就能将明文请求发到 burp 上去了。

```
def toburp(data):
    proxies = {'http':'http://127.0.0.1:8080'}
    url = 'http://127.0.0.1:8888/test'
    r=requests.post(url,data=data,proxies=proxies)
    return(r.text)
```

最后测试中的效果就像这样，请求中的就是正常通信中的密文解密后的样子，只需要在修改请求中的明文，就可以直接修改密文中对应的内容了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVYicBZmiaQPXI0Gen63ZPfzhGQlTmXc9CXIs81n6LTibw5vPUKHyEIVWn3cAbfMYNPLIuPRbu65t0Ig/640?wx_fmt=png)

最后脱敏后的完整代码见 https://github.com/L3B1anc/simpleencrypt

_**招聘启事**_

安恒雷神众测 SRC 运营（实习生）  
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

设计师（实习生）

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
3、有品牌传播、产品设计或新媒体视觉工作经历；  
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
岗位：Web 安全 安全研究员  
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

岗位：安全红队武器自动化工程师  
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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JU8KTRDias8PibAecVuiangc54zeIvj23WxYBCmlag3yvVckP4Ih0RbW1TUticse3xabNw9ibEAccicD0rQ/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JU8KTRDias8PibAecVuiangc5499PvetBuicaUMFcXyWlAdzlQp8ISllDuH0WgV8qYYIXrEoDQ8szQw9w/640?wx_fmt=jpeg)