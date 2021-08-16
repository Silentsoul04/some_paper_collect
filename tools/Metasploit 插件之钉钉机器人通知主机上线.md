\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/4I6FzuuRCTULDgqV-0QSJA)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**前言**_

当有新的主机上线时，希望可以及时得收到通知，Metasploit 官方本来就有一个邮件会话上线通知的插件 plugins/session\_notifier.rb，但是这需要发件服务器，也不一定能及时收到，所以就写了一个钉钉机器人通知。  

实现这个需求需要写一个插件，能获取到一个会话上线的事件，按照钉钉的开发文档发送一个请求。

_**No.1  
**_

_**插件编写**_

lib/msf/core/plugin.rb，插件的基类，全部插件都要实现基类的接口，管理控制台调度程序。  

lib/msf/core/plugin\_manager.rb，插件管理器，实现了 load 和 unload 方法  

插件可以通过添加新特性、新的用户界面命令或任何其他任意方法来更改框架的行为。  

官方有一个例子 plugins/sample.rb，里面 ConsoleCommandDispatcher 是实现控制台调度程序的一个类。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85IMFUaf3lwB98Z3fiaCjmjMlr92dEtQG7XtTH5jNuGH1DKDqGCnhj6tYw/640?wx_fmt=png)

```
class ConsoleCommandDispatcher
    include Msf::Ui::Console::CommandDispatcher

    #
    # The dispatcher's name.
    #
    def name
        "Sample"
    end

    #
    # Returns the hash of commands supported by this dispatcher.
    #
    def commands
        {
            "sample" => "A sample command added by the sample plugin"
        }
    end

    #
    # This method handles the sample command.
    #
    def cmd\_sample(\*args)
        print\_line("You passed: # {args.join(' ')}")
    end
end
```

上面的代码功能：添加一个 sample 命令到控制台调度器（help 命令可以看见），cmd\_sample 为这个命令的动作。  

如果你的插件需要控制台操作设置就需要使用到插件基类里面的 add\_console\_dispatcher 方法添加控制台调度程序。

_**_**No.2  
**_**_

_**_**事件通知订阅**_**_

因为我们只需要会话上线的事件，所以只需要在当前的类将 on\_session\_open 方法实现，其他会话事件可以在 lib/msf/core/session.rb 文件中找到。  

实现了 on\_session\_open 方法还不行，因为没有程序调用它，所以还要将它加入 session\_event\_subscribers 会话事件的订阅器。

```
self.framework.events.add\_session\_subscriber(self)
```

事件的调度器可以在 lib/msf/core/event\_dispatcher.rb 找到，还没添加事件订阅是会话的事件订阅器只有两个。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85Ihia0nfPAv06qBpFl7Ttj6B2QDR2RIiavP6kiaeXZicHnc46CeFBzib5nXOw/640?wx_fmt=png)

执行添加订阅器的代码后，将自身类添加到订阅器列表中，现在订阅器长度为 3，说明添加成功。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85IyjZlVPgicTb868zDsu4RiaTlaDhf9LxfvMwnDmDLcQrHLWjIShicuv5EQ/640?wx_fmt=png)

添加后，即将当前类添加到会话事件订阅器，当有会话事件发生时会到 session\_event\_subscribers 这个列表中调用下面的方法。妙啊！

```
\# 不写事件方法名，当方法名找不到是自动调用下面的函数处理全部事件，下面以on\_session\_open事件为例
def method\_missing(name, \*args)
    event,type,rest = name.to\_s.split("\_", 3)  # event => on; type => session; rest => open
    subscribers = "# {type}\_event\_subscribers"  # 得到拼接会话订阅器列表：session\_event\_subscribers
    found = false
    case event
    when "on"
        if respond\_to?(subscribers, true)
            found = true
            self.send(subscribers).each do |sub|
                next if not sub.respond\_to?(name, true)  # 我们在写插件时有定义on\_session\_open这个方法，当然不会跳过
                sub.send(name, \*args)  # 通过反射判断session\_event\_subscribers的类中有没有on\_session\_open这个方法，用就调用
            end
        else
            (general\_event\_subscribers + custom\_event\_subscribers).each do |sub|
                next if not sub.respond\_to?(name, true)
                sub.send(name, \*args)
                found = true
            end
        end
    when "add"
        if respond\_to?(subscribers, true)
            found = true
            add\_event\_subscriber(self.send(subscribers), \*args)
        end
    when "remove"
        if respond\_to?(subscribers, true)
            found = true
            remove\_event\_subscriber(self.send(subscribers), \*args)
        end
    end
    return found
end
```

_**No.3  
**_

_**发送 Webhook 请求**_

按照钉钉的开发文档发送 markdown 格式的消息，通过响应的错误码判断是否发送成功。

```
def send\_text\_to\_dingtalk(session)
    # https://ding-doc.dingtalk.com/doc# /serverapi2/qf2nxq/9e91d73c
    uri\_parser = URI.parse(dingtalk\_webhook)
    markdown\_text = "## You have a new # {session.type} session!\\n\\n" \\
        "\*\*platform\*\* : # {session.platform}\\n\\n" \\
        "\*\*tunnel\*\* : # {session.tunnel\_to\_s}\\n\\n" \\
        "\*\*arch\*\* : # {session.arch}\\n\\n" \\
        "\*\*info\*\* : > # {session.info ? session.info.to\_s : nil}"
    json\_post\_data = JSON.pretty\_generate({
        msgtype: 'markdown',
        markdown: { title: 'Session Notifier', text: markdown\_text }
        })
    http = Net::HTTP.new(uri\_parser.host, uri\_parser.port)
    http.use\_ssl = true
    request = Net::HTTP::Post.new(uri\_parser.request\_uri)
    request.content\_type = 'application/json'
    request.body = json\_post\_data
    res = http.request(request)
    body = JSON.parse(res.body)
    print\_status((body\['errcode'\] == 0) ? 'Session notified to DingTalk.' : 'Failed to send notification.')
end
```

_**No.3  
**_

_**使用演示**_

创建钉钉机器人，设置关键词：session

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85I6ZwCyVibicZLXXXHQrFDicvkuIE0vwWIpzB7U1G4YjPq3MicSn5UHkbj5w/640?wx_fmt=png)

```
msf6 exploit(multi/handler) > load session\_notifier 
\[\*\] Successfully loaded plugin: SessionNotifier
msf6 exploit(multi/handler) > set\_session\_dingtalk\_webhook https://oapi.dingtalk.com/robot/send?access\_token=5a439cc0009abd551a97e1302a964801da2f3ffe5ba06e97d19294a55202caa3
msf6 exploit(multi/handler) > start\_session\_notifier 
\[\*\] DingTalk notification started.
msf6 exploit(multi/handler) > run

\[\*\] Started reverse TCP handler on 192.168.56.1:7788 
\[\*\] Sending stage (175174 bytes) to 192.168.56.105
\[\*\] Meterpreter session 1 opened (192.168.56.1:7788 -> 192.168.56.105:1078) at 2020-10-04 11:08:54 +0800
\[\*\] Session notified to DingTalk.

meterpreter >
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85InHpoxibReIyYsAuiaKdvP6D6nQMj8DV8bwx9wFUWGa8BR2Qn5AsbE0MA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85IFhSaJbK9ibdDQcfvUCC0XkwdU5BsY62UznIhAxU5sZdvLaFcHUO6oicQ/640?wx_fmt=png)

参考：

https://github.com/rapid7/metasploit-framework/pull/13571

_**招聘**_

安恒雷神众测 SRC 运营（实习生）  
————————  
【职责描述】  
1\.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2\.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3\.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4\.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5\.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

 4.  具备美术功底、懂得设计美化等

简历投递至 bountyteam@dbappsecurity.com.cn

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

简历投递至 bountyteam@dbappsecurity.com.cn

岗位：红队武器化 Golang 开发工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1\. 负责红蓝对抗中的武器化落地与研究；  
2\. 平台化建设；  
3\. 安全研究落地。  
【岗位要求】  
1\. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2\. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3\. 了解 docker，能进行简单的项目部署；  
3\. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4\. 熟悉 TCP/IP 协议的基本运作原理；  
5\. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1\. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2\. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4\. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5\. 具备良好的英语文档阅读能力。  
简历投递至 bountyteam@dbappsecurity.com.cn

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
1\. 定期面向部门、全公司技术分享;  
2\. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3\. 负责完成部门渗透测试、红蓝对抗业务;  
4\. 负责自动化平台建设  
5\. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1\. 至少 1 年安全领域工作经验；  
2\. 熟悉 HTTP 协议相关技术  
3\. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4\. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5\. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6\. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7\. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8\. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1\. 具备良好的英语文档阅读能力；  
2\. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3\. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4\. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5\. 开发过安全相关的开源项目；  
6\. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7\. 个人技术博客；  
8\. 在优质社区投稿过文章；

岗位：安全红队武器自动化工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1\. 负责红蓝对抗中的武器化落地与研究；  
2\. 平台化建设；  
3\. 安全研究落地。  
【岗位要求】  
1\. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2\. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4\. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5\. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1\. 有高并发 tcp 服务、分布式等相关经验者优先；  
2\. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4\. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5\. 具备良好的英语文档阅读能力。

简历投递至 bountyteam@dbappsecurity.com.cn

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85IVZRRcxelxw5EQbOHXPXkuyMxzxFZaYDyndCJupYNhPXqZHfrfHHfaQ/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUCsOA6BXoUpJvJh4Hyq85Iz4vEu4SiaatnQHvAPdXgiafvauxMdHrh3MibQd8rMGO109OxygNct3hiaw/640?wx_fmt=jpeg)