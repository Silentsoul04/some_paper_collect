> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kzLAWKs23vTLeV0a0RreYw)

本文是我在探索和学习 ATT&CK 框架过程中，搭建本地实验环境的粗略过程。如果你也是跟着 Elastic Stack 官方文档无法搭建完成，不如回头翻一下我的这篇文章。

ATT&CK 框架是什么相信大家都比较熟悉了这里我不做介绍了，选择部署 Elastic 安全的原因有两个，一是在做红蓝对抗研究时缺少一款 EDR 产品用作行为分析和 Bypass 技术研究，二是 Elastic 是唯一参加过 ATT&CK 官方的评估而且可以免费使用的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UUG1Agjj0Ex1n0HFOklBt5FeEVN9iaibGbx0QanVA85uAT0ib9pSOd8GVg/640?wx_fmt=jpeg)

**Elastic Security 介绍**
-----------------------

Elastic Security 在一个解决方案中将 SIEM 威胁检测功能与端点防护和响应功能结合在一起。这些分析和保护功能被 Elasticsearch 的速度和可扩展性所利用，使分析人员能够在损失和损失发生之前保护组织免受威胁。

Elastic Security 提供以下安全优势和功能：

• 用于识别攻击和系统错误配置的检测引擎 • 用于事件分类和调查的工作空间 • 交互式可视化以调查过程关系 • 内置的案件管理和自动操作 • 使用预建的机器学习异常作业和检测规则检测无特征的攻击

Elastic Security 助力分析师防御、检测威胁，并就威胁做出响应。免费、开放的解决方案提供了 SIEM、Endpoint Security、威胁搜寻、云监测等功能。Elastic 安全分为 SIEM 和 Endpoint 安全两个大 “功能”。Elastic 的终端安全方案主要由 Elastic Agent（终端代理）、Elasticsearch（搜索和分析引擎）和 Kibana（数据进行可视化）三大件。Elastic Agent 是一种向所有主机添加对日志，指标和其他类型数据的监视的统一方法。单个代理使在整个基础架构中部署监视变得更加轻松快捷。代理程序的单一统一策略使添加新数据源的集成变得更加容易。Fleet 在 Kibana 中提供了一个基于 Web 的 UI，以添加和管理流行服务和平台的集成，以及管理 Elastic Agent 团队。集成提供了一种轻松的方法来添加新的数据源，此外，它们还附带了现成的资产，如仪表板，可视化效果和管道，可从日志中提取结构化字段。这使得在几秒钟内获得洞察变得更加容易。Elastic Security 是 Kibana 的内置部分。要使用 Elastic Security，您只需要部署一个 Elastic Stack（一个 Elasticsearch 集群和 Kibana）。

Elastic Security 组件和工作流程
------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UmotZhnDBhtDJeLLjKC9rql8XC6KtyhOLwDp5BicVwxSwZ3norZMHoJg/640?wx_fmt=png) 

数据通过 Beat 模块和 Elastic Endpoint Security 代理集成从您的主机传送到 Elasticsearch：  

•Elastic Endpoint Security - 弹性代理集成，可保护您的主机免受恶意软件的侵害并提供以下数据集：

•Windows：进程，网络，文件，DNS，注册表，DLL 和驱动程序负载，恶意软件安全检测 •Linux / macOS：进程，网络，文件

•Beat 模块：Beat 是轻量级的数据托运者。Beat 模块提供了一种从常见来源（例如云和操作系统事件，日志和指标）收集和解析特定数据集的方法。

Kibana 中的 Elastic Security 应用程序用于管理检测引擎、Cases 和 Timeline，以及管理运行 Endpoint Security 的主机：

• 检测引擎：通过以下方式自动搜索可疑的主机和网络活动：

• 检测规则：定期搜索主机发送的数据（Elasticsearch 索引）以查找可疑事件。发现可疑事件时，将生成检测警报。生成警报时，可以使用外部系统（例如 Slack 和电子邮件）来发送通知。您可以创建自己的规则并使用我们的预建规则。• 例外：减少噪音并减少误报次数。异常与规则相关联，并在满足异常条件时阻止发出警报。值列表包含可用作异常条件一部分的源事件值。在主机上安装 Elastic Endpoint Security 后，您可以从 Security 应用程序将恶意软件例外直接添加到端点。• 机器学习作业：自动异常检测主机和网络事件。每个主机都提供异常分数，并且可以与检测规则一起使用。• 时间轴：用于调查警报和事件的工作区。时间轴使用查询和过滤器来深入分析与特定事件相关的事件。时间线模板附加到规则，并在调查警报时使用预定义的查询。时间线可以保存并与其他人共享，也可以附加到案例中。• 案例：直接在 Security 应用程序中打开，跟踪和共享安全问题的内部系统。案例可以与外部票务系统集成。• 管理：查看和管理运行 Elastic Endpoint Security 的主机。

开启 Fleet
--------

Elasticsearch（以下简称 es）和 Kibana 的安装非常简单，此处省略…… 下面主要记录如何开启 Fleet 并配置 SSL/TLS。

证书生成
----

使用 Elast Agent 需要开启 SSL，为 ES 生成私钥和 X.509 证书。其他的 Beats 程序（filebeat、winlogbeat 等）也都需要添加相应的 SSL 配置。

1. 同时生成 CA 和服务器证书

`elasticsearch-certutil.bat cert ca --pem --ip 10.0.0.X --name master --keep-ca-key --out certs.zip`

1.crt 转换成 pem 格式：

`openssl x509 -in ca.crt -out ca.pem`

其他 Beats 只需要 ca.pem 即可。Windows 系统中安装 ca.crt 证书时手动选择将证书导出受信任的根颁发机构。Linux 系统中将 ca.pem 复制到 / etc/ssl/certs / 目录下，运行 update-ca-trust 或 update-ca-certificates。

妥善保管 ca.key，再次生成证书时需要。

1.elasticsearch.yml 配置增加如下：

```
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.key: certs/master.key
xpack.security.http.ssl.certificate: certs/master.crt
xpack.security.http.ssl.certificate_authorities: certs/ca.crt
xpack.security.transport.ssl.key: certs/master.key
xpack.security.transport.ssl.certificate: certs/master.crt
xpack.security.transport.ssl.certificate_authorities: certs/ca.crt
```

1.kibana.yml 配置修改 elasticsearch.ssl.certificateAuthorities 如下：

```
elasticsearch.ssl.certificateAuthorities: [ "D:\\elk\\kibana-7.10.1-windows-x86_64\\config\\certs\\ca.crt" ]
```

生成密码
----

生成密码前，需要已经启动 ES。

```
PS D:\ELK\elasticsearch-7.10.1\bin> .\elasticsearch-setup-passwords.bat auto
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y

Changed password for user apm_system
PASSWORD apm_system = VfD5gOgdoJu3AeQCEk52

PASSWORD kibana_system = bo7AkvpKixyU9wOzZ9ob

Changed password for user kibana
PASSWORD kibana = bo7AkvpKixyU9wOzZ9ob

Changed password for user logstash_system
PASSWORD logstash_system = I8D0VG5H1csHDatR7bmB

Changed password for user beats_system
PASSWORD beats_system = Tyd1zP7T7a8WIF3fpZiR

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = Kn77JNDaNuUMyvBOOfxI

Changed password for user elastic
PASSWORD elastic = wRdchJt8yNAOX8WHdCE6    已改为password
```

kibana-7.10.1 增加额外的配置，kibana.yml：

```
elasticsearch.hosts: ["https://10.0.0.X:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "bo7AkvpKixyU9wOzZ9ob"

xpack.fleet.enabled: true
xpack.fleet.agents.tlsCheckDisabled: true 
xpack.encryptedSavedObjects.encryptionKey: "something_at_least_32_characters"
xpack.security.enabled: true
```

Beats 配置
--------

winlogbeat、Filebeat 等 yaml 格式的配置文件一般只需要添加或许该如下配置：

```
output.elasticsearch:
  hosts: ["10.0.0.2:9200"]
  protocol: "https"
  username: "elastic"
  password: "password"
  ssl.certificate_authoritoes: "/home/ubuntu/ca/ca.pem"
```

安装 Elastic Agent
----------------

Elastic Agent 安装和注册已包含在 Fleet 中但是该功能目前处于 Bata 版本，官方文档也并不全。在 agent 的配置文件中写 ssl 配置根本不生效，好像代码还没有实现，而 golang 写的程序默认与系统的证书信任关系有关。所以在 Windows 上必须导入生成的 ca.crt 证书，必须是导入此计算机，不能是此用户。

添加 Agent
--------

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UzknvINwfcMicH9FxvSOtOMmEkibvAufLS3oRj1ySLZgsaWnWykdiawCqA/640?wx_fmt=png)

从 Fleet 页面添加 agent，自动生成注册命令。将 elastic-agent 安装包复制到目标系统，运行注册命令。  

`.\elastic-agent.exe install -f --kibana-url=http://10.0.0.X:5601 --enrollment-token=d3dUQV9IWUJIOVlqUjN6OF9Za0k6MkJLYS1MQzhRSXViZzh5QmhhU3JlQQ==`

确认 Agent 状态
-----------

需要确认三处：1. Status 是 Online，2. Data streams 有数据，3. es 的控制台实时日志没有报错。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UibxgN1bI4Xpgyiad6gA02y5vDXjFpMvzd2NQxesa13ArmSIbmCvylicVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UzljiaBnL2fCTqqGrVdPCEicribDuL8QiaJkrr1E1KnARrXa6ODQmaKB0Nw/640?wx_fmt=png)   如果出现 bad_certificate 错误，再检查证书生成和安装的步骤和过程。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UiaWhPlzeaRxx4DJDVVyYQm2TWicwaUhU6xHJklrVkADQHdIfaDwF4ZHQ/640?wx_fmt=png)

查看 Security/Overview 是否有数据。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UlXU6KJyeqC1xKH4yGsL2q7YWGdsvqM4TdeMAPFCbzLrp7qr56YpTjA/640?wx_fmt=png)

开启检测规则
------

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UM6Wgt7NpGvG8KtVLrArYA16KjmvUbHg6X6IELbtticfWCPAadVv6Tew/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UYH3icyOMtCe6LdMdIvAALrIYicIG9YweR73BUgvT6mE9KqtFAzwaX5lQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UVfCPNlLrp8y4dY6hSHFEIrOnM3t8LBfNlR0FM0Vy1rNPVGNOFqMWpg/640?wx_fmt=png)

Elastic Security 的检测引擎规则已在 GitHub 上开源。开启规则需要手动点击每个要开启的规则，还好规则不多。

使用 Elastic
----------

Elastic Agent 默认收集以下数据源：

•Process - Linux, macOS, Windows•Network - Linux, macOS, Windows•File - Linux, macOS, Windows•DNS - Window•Registry - Windows•DLL and Driver Load - Windows•Security - Windows

要用主机和网络安全事件发送到 Elastic Security，需要在要从中提取安全事件的主机上安装和配置 Beats：

•Filebeat 用于转发和集中日志和文件 •Auditbeat 用于收集安全事件 •Winlogbeat 用于集中 Windows 事件日志 •Packetbeat 用于分析网络活动

可以使用 Kibana UI 或直接从命令行安装 Beats。

代理策略
----

使用代理策略管理代理及其收集的数据。不同的操作系统可以使用集成应用不同，所以应创建不同的代理策略以应用于不同的系统。

新建一个命名为 Windows 的策略，添加 “Endpoint Security”，“windows” 集成，安装并配置 Sysmon。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UuJcxOT0uLicKAXdXkfoWw9shbxCa3ia32zr3NAPyjA1QE9L54FUmF2GQ/640?wx_fmt=png)

一个针对 Windows 系统的策略已配置完成。将策略下发给代理即可。

后续
--

在搭建完毕基本环境后，我们将在此环境中进行 ATT&CK 官方指导的 APT29 对手模拟。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UpC1JBnQVuXupufBSHGgnXqp2omLIXOhM7Y7Q4E2z7q8zBaqmickqmrA/640?wx_fmt=jpeg)

参考链接
----

•https://www.elastic.co/guide/en/elasticsearch/reference/7.10/configuring-tls.html#node-certificates•https://www.elastic.co/guide/en/fleet/7.10/fleet-quick-start.html•https://www.elastic.co/cn/blog/configuring-ssl-tls-and-https-to-secure-elasticsearch-kibana-beats-and-logstash•https://github.com/elastic/kibana/issues/73483•https://www.elastic.co/cn/downloads/elastic-agent•https://www.elastic.co/guide/en/fleet/current/index.html•https://github.com/elastic/detection-rules

**关于山石网科**

  

  

  

山石网科是中国网络安全行业的技术创新领导厂商，自成立以来一直专注于网络安全领域前沿技术的创新，提供包括边界安全、云安全、数据安全、内网安全在内的网络安全产品及服务，致力于为用户提供全方位、更智能、零打扰的网络安全解决方案。山石网科为金融、政府、运营商、互联网、教育、医疗卫生等行业累计超过 18,000 家用户提供高效、稳定的安全防护。山石网科在苏州、北京和美国硅谷均设有研发中心，业务已经覆盖了中国、美洲、欧洲、东南亚、中东等 50 多个国家和地区。