> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DVC5vSNIZCpevGBjvxNp4Q)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**前言**_

●在进入目标主机进行信息收集时会收集到各种密码凭证，Metasploit 提供了管理从目标主机上收集到的凭证的接口。前提是需要开启 msfdb 服务。

●所以需要初始化 postgres 数据库，初始化数据库在 metasploit 官方的 issues 上已经是一个经久不衰的话题，非常多人遇到了问题，下面来填了数据库初始化这个坑。

_**No.2  
**_

_**初始化数据库**_

●安装 postgresql，各个系统的安装方式不一样，有些系统会在安装完成后执行钩子程序添加 postgres 用户，有些系统就没有，请自行添加。  

●先添加 postgres 用户密码 passwd postgres 输入两遍密码，su - postgres -c "initdb --locale en_US.UTF-8 -D'/var/lib/postgres/data'"，输入之前设置的 postgres 密码。

●先别急着初始化 msfdb，数据库默认使用 /var/run/postgres 作为套接字的目录，普通用户没有权限访问。

●取消

/usr/share/postgresql/postgresql.conf.sample 文件里面的 unix_socket_directories = '/tmp' # comma-separated list of directories 注释。

●重新打开终端执行初始化命令，执行 export PGHOST=/tmp 设置环境变量，重启 postgres 数据库服务 sudo systemctl restart postgresql.service。

●执行./msfdb reinit 初始化 msfdb 数据库服务，没什么意外的话，应该会出现下面的界面。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FbFYbPF7AJSPOSaicgsINrklvPfowqDy6fbhicokiaVgYZeBvL76mkEpUw/640?wx_fmt=png)

_**No.3  
**_

_**开发模板**_

```
def report_store_config(config)
    service_data = {
        address: config[:hostname], # 就是服务的IP地址
        port: config[:port], # 服务开放的端口
        service_name: 'smb', # 服务的名称
        protocol: 'tcp', # 服务连接的协议
        workspace_id: myworkspace_id # 这个不用改（默认）
        }

    credential_data = {
        origin_type: :session, # 来源类型，cong目标session得到的就是当前session的实例化对象
        session_id: session_db_id, # session的id
        post_reference_name: refname, # 当前执行的模块名称，就是你用哪一个模块获取的信息
        private_type: :password, # 密码类型，是明文密码还是，hash密文
        private_data: config[:password], # 密码，字符串，一定要UTF-8编码，不然进数据库会报错
        username: config[:username] # 用户名
        }.merge(service_data)

    credential_core = create_credential(credential_data)

    login_data = {
        core: credential_core,
        status: Metasploit::Model::Login::Status::UNTRIED # 密码登录的状态，下面详细说明
        }.merge(service_data)

    create_credential_login(login_data)
end
```

_**No.4  
**_

_**metasploit 凭证 ER 图**_

●如果只看凭证这块的话可以从 metasploit_credential_logins 这个表看起，上面代码 service_data 就是创建 services 表的，如凭证中的服务没有会自动创建服务，credential_data 则是创建凭证需要用到的。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FibnGMmVoodib78lIO3OhmfmRLQPIQmTDSyziadjjDx5lfoBDpiciavHmWow/640?wx_fmt=png)

**service_data**

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0Ftj8Dia3icptCpMx9BKXICDgNQPdgDvRjBH0UXELSMNsjGOTKTg2QFluw/640?wx_fmt=png)

●凭证的来源

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FK9V5AMedFicF06Hw9tttplhlApYBM4GcbqeVOGm1Tn4XSASPT2vJ2JQ/640?wx_fmt=png)

**credential_data**

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FPmIBF5PK4meHJ3YiczMzVUozI3EUL7JUYGwm8GBVwrgH8t82Tg1gmbQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0F4sppmkRHfkhFOnT6kh3QL5J95kQia2icZ0AqCMvcJytvyjvqDiaibrobsw/640?wx_fmt=png)

●帐号密码都是分表存储的，为什么要这样呢？，为了方便 auxiliary 模块从数据库中获取用户名和密码进行爆破服务，简单来说一个表就是一个字典。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FzRpbRcG3gqIDXKDmegdZIXicpbDRqNPK2rH2Ogia3Bj8PpqpXD14pwPA/640?wx_fmt=png)

**login_data**

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0Ficu9NvP920icDzibv8sZsCeibyorEWfIViaBqD3FcExJUpYhicTlWfiaTbjCw/640?wx_fmt=png)

_**No.5  
**_

_**使用示例**_

●下面是 hashdump 和 kiwi 密码 hash 入库代码片段，通过将收集到的信息导进库里面，在控制台输入 creds 命令可查看全部凭证，并且可以通过搜索指定的 IP 或者 IP 段显示密码凭证，也可以筛选制定服务类型或者密码类型的凭证。在也不用往上找之前收集的密码了。

```
def report_creds(user_data)
    user = user_data.user_name
    pass = "# {user_data.lanman}:# {user_data.ntlm}"
    return if (user.empty? || pass.include?('aad3b435b51404eeaad3b435b51404ee'))

    # Assemble data about the credential objects we will be creating
    credential_data = {
        origin_type: :session,
        post_reference_name: 'hashdump',
        private_data: pass,
        private_type: :ntlm_hash,
        session_id: client.db_record.id,
        username: user,
        workspace_id: shell.framework.db.workspace.id
        }

    credential_core = shell.framework.db.create_credential(credential_data)

    # Assemble the options hash for creating the Metasploit::Credential::Login object
    login_data = {
        core: credential_core,
        status: Metasploit::Model::Login::Status::UNTRIED,
        address: ::Rex::Socket.getaddress(client.sock.peerhost, true),
        port: 445,
        service_name: 'smb',
        protocol: 'tcp',
        workspace_id: shell.framework.db.workspace.id
        }

    shell.framework.db.create_credential_login(login_data)
end
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FZr7ZFX8NCGEYRia9xrxOv1iaky1Mqr9KJA96g42mU5sL79sMsIPjotsQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FHxIaK6HiahkYbdqIWdicPicgiaDThG3Uias0CV2EhTqxeTMW3DaKsnTGxzA/640?wx_fmt=png)

●开启 Metasploit 的数据库服务可以保存你的操作记录，就算把控制台退出了或者 session 掉线了。之前的操作过命令和输出都会保存在数据库里。

●如果你在收集信息的时候无法将密文解密，可以先保存到数据库，只要你还知道算法，在入库的时候提交 jtr_format，后面可以调用 hashcat 和 John the ripper 进行自动化的字典攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0Fick1HpKEcdOiaO4DsmDSG1cZYic8icU9CDg96h48Iw9ZScfFYXWnVqDPicg/640?wx_fmt=png)

●重要的是 Metasploit 还提供 API 接口，这使得自动化和其他工具联动成为了可能。

_**No.6  
**_

_**参考**_

https://github.com/rapid7/metasploit-framework/pull/14432  
https://github.com/rapid7/metasploit-framework/blob/master/documentation/api/v1/credential_api_doc.rb

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JXv8rHUicrKB8yvnwltZGI0FBWSJ1LXQIPQOoZBWiabibVkPj7S837zo1Dia9hSXRJpYZ17IDGwJn0ESw/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JViaPPu2Frsh8IEjvWRD0gGUjMPyFmEuKL26LBRkp2Ke3Cdicag9EJQddFOBZXuy2nbGGZMQibFMVibOg/640?wx_fmt=jpeg)