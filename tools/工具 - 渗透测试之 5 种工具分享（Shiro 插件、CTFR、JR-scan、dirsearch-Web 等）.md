> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pQJfOlxWkUOnzfaYKaVR-A)

![](https://mmbiz.qpic.cn/mmbiz_gif/GzdTGmQpRic1kdT5lkh0tP2ibZUf0k2NXUFYS9OVMkmYUnIGxe32uCkz8kUdsia9A1aHQuiaW0FKAsk0zE5Me4ERzQ/640?wx_fmt=gif)

工具目录  

-------

1.BurpShiroPassiveScan 是一款基于 BurpSuite 的被动式 shiro 检测插件 2.reconftw 是对具有多个子域的目标执行全面检查的脚本 3.CTFR 是一款不适用字典攻击也不适用蛮力获取的子域名的工具 4.JR-scan 是一款一键实现基本信息收集，支持 POC 扫描，支持利用 AWVS 探测的工具 5.dirsearch-Web 是一种成熟的命令行工具，旨在暴力破解 Web 服务器中的目录和文件

1.BurpShiroPassiveScan
----------------------

### 介绍

BurpShiroPassiveScan 一个希望能节省一些渗透时间好进行划水的扫描插件

该插件会对 BurpSuite 传进来的每个不同的域名 + 端口的流量进行一次 shiro 检测

目前的功能如下

•shiro 框架指纹检测 •shiro 加密 key 检测

### 安装

这是一个 java maven 项目

如果你想自己编译的话, 那就下载本源码自己编译成 jar 包 然后进行导入 BurpSuite

如果不想自己编译, 那么下载该项目提供的 jar 包 进行导入即可

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEXHTzQHQZy0xWej7UvllW9a8icBVugIYJT6sQJQ24HzefibegpzGKBaZw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEjyxalCXLXpGN0lukV68KLDTxP7wGvXITicIgSY5yrcMzK1ic9n37tCeA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEFYPHmIbySWpibXm61gBCoibQKthfqWvDBz1HTqTIXTvkaZUWqjicyUUTg/640?wx_fmt=png)

### 检测方法选择

目前有一种方法进行 shiro 框架 key 的检测

1.l1nk3r 师傅 的 基于原生 shiro 框架 检测方法

l1nk3r 师傅的检测思路地址: https://mp.weixin.qq.com/s/do88_4Td1CSeKLmFqhGCuQ

目前这两种方法都已经实现！！！

根据我的测试 l1nk3r 师傅 的更加适合用来检测 “shiro key” 这个功能！！！

使用 l1nk3r 师傅 这个方法 对比 URLDNS 我认为有以下优点

1. 去掉了请求 dnslog 的时间, 提高了扫描速度, 减少了大量的额外请求 2. 避免了有的站点没有 dnslog 导致漏报 3. 生成的密文更短, 不容易被 waf 拦截

基于以上优点, 我决定了, 现在默认使用 l1nk3r 师傅 这个方法进行 shiro key 的爆破

### 使用方法

例如我们正常访问网站

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEpCpuzQwpfQAeMAWibEJkOusIYcDRN77bCEovqP2JYRHxdwkbmxJy1KQ/640?wx_fmt=png)  
访问完毕以后, 插件就会自动去进行扫描

如果有结果那么插件就会在以下地方显示

•Extender•Scanner-Issue activity

### 问题查看

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEgf0xhbQOndla0vWX1l3RYQtXyrTIWkCNsEiaIc1sIFLvn9ic0tJp2DVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoE7WTZEZdECSVibmmQEmLcK1vYkEn6z5erJ5dxUm0OAHcb7K0ibxZ6Id9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEDDkicECX0vuc01zhibzLPRCXIJHiaCzRkYMNhZxSI5FciaYbenghSuAP3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoExx6km9SUCnqSNVeibicFBq23Ca88TjA80kfoyIAjEiaEj2h9cP5wcCooQ/640?wx_fmt=png)  
shiro 加密 key 查看

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEDDkicECX0vuc01zhibzLPRCXIJHiaCzRkYMNhZxSI5FciaYbenghSuAP3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoExx6km9SUCnqSNVeibicFBq23Ca88TjA80kfoyIAjEiaEj2h9cP5wcCooQ/640?wx_fmt=png)

### shiro 加密方法

目前搭配了两种加密方法 cbc 与 gcm

cbc 就是经常使用的

gcm 就是最新出的

### tag 界面查看漏洞情况

现在可以通过 tag 界面查看漏洞情况了

分别会返回

•waiting for test results = 扫描 shiro key 中 •shiro key scan out of memory error = 扫描 shiro key 时, 发生内存错误 •shiro key scan diff page too many errors = 扫描 shiro key 时, 页面之间的相似度比对失败太多 •shiro key scan task timeout = 扫描 shiro key 时, 任务执行超时 •shiro key scan unknown error = 扫描 shiro key 时, 发生未知错误 •[-] not found shiro key = 没有扫描出 shiro key•[+] found shiro key: xxxxxx = 扫描出了 shiro key

注意: 发生异常错误的时候, 不用担心下次不会扫描了, 下次访问该站点的时候依然会尝试进行 shiro key 扫描, 直到扫描完毕为止

### 项目地址

https://github.com/suifengg/BurpShiroPassiveScan

2.reconftw
----------

### 介绍

这是一个简单的脚本，旨在对具有多个子域的目标执行全面检查。

### 安装

```
git clone https://github.com/six2dez/reconftwcd reconftwchmod +x *.sh./install.sh./reconftw.sh -d target.com -a
```

### 用法

./reconfw.sh -h

```
$ ./reconftw.sh -h  ██▀███  ▓█████  ▄████▄   ▒█████   ███▄    █   █████▒▄▄▄█████▓ █     █░ ▓██ ▒ ██▒▓█   ▀ ▒██▀ ▀█  ▒██▒  ██▒ ██ ▀█   █ ▓██   ▒ ▓  ██▒ ▓▒▓█░ █ ░█░                                                ▓██ ░▄█ ▒▒███   ▒▓█    ▄ ▒██░  ██▒▓██  ▀█ ██▒▒████ ░ ▒ ▓██░ ▒░▒█░ █ ░█                                                 ▒██▀▀█▄  ▒▓█  ▄ ▒▓▓▄ ▄██▒▒██   ██░▓██▒  ▐▌██▒░▓█▒  ░ ░ ▓██▓ ░ ░█░ █ ░█                                                 ░██▓ ▒██▒░▒████▒▒ ▓███▀ ░░ ████▓▒░▒██░   ▓██░░▒█░      ▒██▒ ░ ░░██▒██▓                                                 ░ ▒▓ ░▒▓░░░ ▒░ ░░ ░▒ ▒  ░░ ▒░▒░▒░ ░ ▒░   ▒ ▒  ▒ ░      ▒ ░░   ░ ▓░▒ ▒                                                    ░▒ ░ ▒░ ░ ░  ░  ░  ▒     ░ ▒ ▒░ ░ ░░   ░ ▒░ ░          ░      ▒ ░ ░                                                    ░░   ░    ░   ░        ░ ░ ░ ▒     ░   ░ ░  ░ ░      ░        ░   ░                                                     ░        ░  ░░ ░          ░ ░           ░                      ░                                                                    ░                                                                                                                             by @six2dez1(Twitter) or @six2dez(rest of sites)                                                Params (-d always required):        ./reconftw.sh -d target.com     Target domain (required always)        ./reconftw.sh -l targets.txt    Web list (required only with -w) Flags (1 required):         ./reconftw.sh -a        All checks (default and recommended)        ./reconftw.sh -s        Only subdomains        ./reconftw.sh -g        Only Google Dorks        ./reconftw.sh -w        Only web scan        ./reconftw.sh -h        Show this help Examples:  ./reconftw.sh -d target.com -a -> All checks ./reconftw.sh -d target.com -s -> Only subdomains ./reconftw.sh -d target.com -g -> Only Google Dorks ./reconftw.sh -d target.com -l targets.txt -w -> Only Web Scan (Target list required)
```

### 用例

```
./reconfw.sh -a baidu.com
```

### 特征

Google Dorks（基于 deggogle_hunter）  
子域枚举（多种工具：Pasive，分辨率，蛮力和排列）  
Sub TKO（副翼和核）  
探测（httpx）  
Web 屏幕截图（水色）  
模板扫描仪（核）  
端口扫描（naabu）  
网址提取（waybackurls 和 gau）  
模式搜索（gf 和 gf 模式）  
参数发现（paramspider 和 arjun）  
XSS（Gxss 和 dalfox）  
GitHub 检查（git-hound）  
Favicon Real IP（收藏夹）  
Javascript 检查（JSFScan.sh）  
目录模糊 / 发现（dirsearch 和 ffuf）  
Cors（CORScanner）  
SSL 检查（testssl）

您也可以只执行子域扫描，网络扫描或 Google Dork。请记住，webscan 需要带有 - l 标志的目标列表。

它使用目标域的名称在 Recon / 文件夹中生成并输出，例如 Recon / target.com /

### 项目地址

https://github.com/six2dez/reconftw

3.CTFR
------

### 介绍

您会错过 AXFR 技术吗？此工具允许您在几秒钟内从 HTTP S 网站获取子域。  
这个怎么运作？CTFR 既不使用字典攻击也不使用蛮力，而只是滥用证书透明度日志。

有关 CT 日志的详细信息，请 www.certificate-transparency.org 和 crt.sh。[1]

### 入门

请按照以下说明安装运行 CTFR

### 先决条件

R 确保安装以下工具

```
Python 3.0 or later.pip3 (sudo apt-get install python3-pip).
```

### 正在安装

```
$ git clone https://github.com/UnaPibaGeek/ctfr.git$ cd ctfr$ pip3 install -r requirements.txt
```

### Running

```
$ python3 ctfr.py --help<br style="box-sizing: border-box;">
```

### 用法

```
-d --domain [target_domain] (required)-o --output [output_file] (optional)
```

### 例子

```
$ python3 ctfr.py -d starbucks.com
```

```
$ python3 ctfr.py -d facebook.com -o /home/shei/subdomains_fb.txt
```

### 项目地址

https://github.com/UnaPibaGeek/ctfr

4.JR-scan
---------

### 介绍

利用 python3 写的综合扫描工具，可 “一键” 实现基本信息收集（端口、敏感目录、WAF、服务、操作系统），支持 POC 扫描（可自行添加 POC，操作简单），支持利用 AWVS 探测，未来争取实现 xray 联动。  
在启动扫描器后，傻瓜式操作即可完成扫描。  
扫描器允许进行单个扫描，批量扫描 (从文件列表里扫描网站)，C 段扫描  
启动方法：直接利用 Python3 运行 JR.py 即可  
提示：最好是在 liux 环境下运行，win 的话，可能会出现编码问题！！！

### 安装

```
git clone https://github.com/suifengg/JR-scanpython3.8 /JR/JR.PY
```

### 启动界面

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoErW9VJrI8fm1O67eBkVgV8wS21IaSKvoDLaWLLVkbT1yGbozNyO7BkQ/640?wx_fmt=png)

### 数据库界面

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEqumwsmUQZXc4b6GIrqBVglKxA6ica54eiaOy8eiav16C9eWGhX9FOAIzw/640?wx_fmt=png)

### 网站整体界面

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEGuN6zNHJB7HqpqtNicJnQ8icV7dcNZdOaeicOboJwqpMm9CxDTibGJgHjw/640?wx_fmt=png)

### 端口界面

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEyG4O1hsFPbTHrsbYHia27dRb1j1RmcFsDQ8uXNJ0p03HGzllegFyy4w/640?wx_fmt=png)

### URL 界面

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoEYBicZGicvlwESR9aYI6XcfiaOd6dCFZYUNaRlfqu8iabsdbeF2VWo2p6SA/640?wx_fmt=png)

### 漏洞界面

![](https://mmbiz.qpic.cn/mmbiz_png/eR1JusQTlickcsOfS94OOQyfHdHKE0NoES4PPEDfvRT6IEuoMqUaexYWsCmhyUEzZ3FlbYuDpibALZde0kHL1lwQ/640?wx_fmt=png)

### 项目地址

https://github.com/suifengg/JR-scan

5.dirsearch-Web 路径扫描器
---------------------

### 总览

•Dirsearch 是一种成熟的命令行工具，旨在暴力破解 Web 服务器中的目录和文件。• 随着 6 年的增长，dirsearch 现在已成为顶级的 Web 内容扫描仪。• 作为功能丰富的工具，dirsearch 为用户提供了执行复杂的 Web 内容发现的机会，其中包括单词列表的许多矢量，高精度，出色的性能，高级的连接 / 请求设置，现代的蛮力技术和出色的输出。

### 安装及使用

```
git clone https://github.com/maurosoria/dirsearch.gitcd dirsearchpython3 dirsearch.py -u <URL> -e <EXTENSIONS>
```

• 要使用 SOCKS 代理或`../`在单词列表中使用它，您需要使用以下命令安装点子`requirements.txt`：`pip3 install -r requirements.txt`• 如果您使用的是 Windows，并且没有 git，则可以在此处 [2] 安装 ZIP 文件。Dirsearch 还支持 Docker[3]

**Dirsearch 需要 python 3 或更高版本**

### 特征

• 快速 • 易于使用 • 多线程 • 通配符响应过滤（无效的网页）• 保持活跃的联系 • 支持多种扩展 • 支持每种 HTTP 方法 • 支持 HTTP 请求数据 • 扩展不包括 • 报告（纯文本，JSON，XML，Markdown，CSV）• 递归暴力破解 •IP 范围内的目标枚举 • 子目录暴力破解 • 力扩展 •HTTP 和 SOCKS 代理支持 •HTTP cookie 和标头支持 • 文件中的 HTTP 标头 • 用户代理随机化 • 代理主机随机化 • 批量处理 • 请求延迟 •429 个响应码检测 • 多种单词列表格式（小写，大写，大写）• 文件中的默认配置 • 通过主机名强制请求的选项 • 添加自定义后缀和前缀的选项 • 可以将响应代码列入白名单，支持范围（-i 200,300-399）• 可以将响应代码列入黑名单，支持范围（-x 404,500-599）• 选择按大小排除响应 • 选择排除文字回复 • 选择排除正则表达式的响应 • 选择通过重定向排除响应 • 仅显示响应长度在范围内的项目的选项 • 从每个单词列表条目中删除所有扩展名的选项 • 静音模式 • 调试模式

### 关于词表

摘要：

Wordlist 必须是一个文本文件，每一行都是一个端点。关于扩展名，与其他工具不同，如果不使用该`-f`标志，dirsearch 不会将扩展名附加到每个单词上。默认情况下，仅`%EXT%`单词表中的关键字将被扩展名（`-e <extensions>`）替换。

详细资料：

• 单词表中的每一行都将照此处理，除非使用特殊关键字**％EXT％时**，它将为作为参数传递的每个扩展名（-e | --extensions）生成一个条目。

例：

```
root/index.%EXT%
```

传递扩展名 “asp” 和“ aspx”（`-e asp,aspx`）将生成以下字典：

```
root/indexindex.aspindex.aspx
```

• 对于没有**％EXT％的**单词列表（例如 SecLists[4]），您需要使用 ***-f | --force-extensions** 开关可将扩展名附加到单词表中的每个单词以及 “/”。对于不想强制使用的单词列表中的条目，可以在它们的末尾添加**％NOFORCE％*，以便 dirsearch 不会附加任何扩展名。

例：

```
admin<br style="box-sizing: border-box;">home.%EXT%<br style="box-sizing: border-box;">api%NOFORCE%<br style="box-sizing: border-box;">
```

通过 **-f** / **--force-extensions** 标志（`-f -e php,html`）传递扩展名 “php” 和“ html”将生成以下字典：

```
adminadmin.phpadmin.htmladmin/homehome.phphome.htmlapi
```

**要使用多个单词列表，可以用逗号分隔单词列表。示例：-w wordlist1.txt，wordlist2.txt**

### 选件

```
Usage: dirsearch.py [-u|--url] target [-e|--extensions] extensions [options]Options:  --version             show program's version number and exit  -h, --help            show this help message and exit  Mandatory:    -u URL, --url=URL   Target URL    -l FILE, --url-list=FILE                        URL list file    --cidr=CIDR         Target CIDR    -e EXTENSIONS, --extensions=EXTENSIONS                        Extension list separated by commas (Example: php,asp)    -X EXTENSIONS, --exclude-extensions=EXTENSIONS                        Exclude extension list separated by commas (Example:                        asp,jsp)    -f, --force-extensions                        Add extensions to the end of every wordlist entry. By                        default dirsearch only replaces the %EXT% keyword with                        extensions  Dictionary Settings:    -w WORDLIST, --wordlists=WORDLIST                        Customize wordlists (separated by commas)    --prefixes=PREFIXES                        Add custom prefixes to all entries (separated by                        commas)    --suffixes=SUFFIXES                        Add custom suffixes to all entries, ignore directories                        (separated by commas)    --only-selected     Only entries with selected extensions or no extension                        + directories    --remove-extensions                        Remove extensions in all wordlist entries (Example:                        admin.php -> admin)    -U, --uppercase     Uppercase wordlist    -L, --lowercase     Lowercase wordlist    -C, --capital       Capital wordlist  General Settings:    -r, --recursive     Bruteforce recursively    -R DEPTH, --recursion-depth=DEPTH                        Maximum recursion depth    -t THREADS, --threads=THREADS                        Number of threads    --subdirs=SUBDIRS   Scan sub-directories of the given URL[s] (separated by                        commas)    --exclude-subdirs=SUBDIRS                        Exclude the following subdirectories during recursive                        scan (separated by commas)    -i STATUS, --include-status=STATUS                        Include status codes, separated by commas, support                        ranges (Example: 200,300-399)    -x STATUS, --exclude-status=STATUS                        Exclude status codes, separated by commas, support                        ranges (Example: 301,500-599)    --exclude-sizes=SIZES                        Exclude responses by sizes, separated by commas                        (Example: 123B,4KB)    --exclude-texts=TEXTS                        Exclude responses by texts, separated by commas                        (Example: 'Not found', 'Error')    --exclude-regexps=REGEXPS                        Exclude responses by regexps, separated by commas                        (Example: 'Not foun[a-z]{1}', '^Error$')    --exclude-redirects=REGEXPS                        Exclude responses by redirect regexps or texts,                        separated by commas (Example: 'https://okta.com/*')    --calibration=PATH  Path to test for calibration    --random-agent      Choose a random User-Agent for each request    --minimal=LENGTH    Minimal response length    --maximal=LENGTH    Maximal response length    -q, --quiet-mode    Quiet mode    --full-url          Print full URLs in the output    --no-color          No colored output  Request Settings:    -m METHOD, --http-method=METHOD                        HTTP method (default: GET)    -d DATA, --data=DATA                        HTTP request data    -H HEADERS, --header=HEADERS                        HTTP request header, support multiple flags (Example:                        -H 'Referer: example.com' -H 'Accept: */*')    --header-list=FILE  File contains HTTP request headers    -F, --follow-redirects                        Follow HTTP redirects    --user-agent=USERAGENT    --cookie=COOKIE  Connection Settings:    --timeout=TIMEOUT   Connection timeout    --ip=IP             Server IP address    -s DELAY, --delay=DELAY                        Delay between requests    --proxy=PROXY       Proxy URL, support HTTP and SOCKS proxies (Example:                        localhost:8080, socks5://localhost:8088)    --proxy-list=FILE   File contains proxy servers    --matches-proxy=PROXY                        Proxy to replay with found paths    --max-retries=RETRIES    -b, --request-by-hostname                        By default dirsearch requests by IP for speed. This                        will force requests by hostname    --exit-on-error     Exit whenever an error occurs    --debug             Debug mode  Reports:    --simple-report=OUTPUTFILE    --plain-text-report=OUTPUTFILE    --json-report=OUTPUTFILE    --xml-report=OUTPUTFILE    --markdown-report=OUTPUTFILE    --csv-report=OUTPUTFILE
```

注意：您可以通过编辑 **default.conf** 文件来更改 dirsearch 的默认配置（默认扩展名，超时，单词列表位置等）。

### 使用

### 简单实用

```
python3 dirsearch.py -u https://targetpython3 dirsearch.py -e php,html,js -u https://targetpython3 dirsearch.py -e php,html,js -u https://target -w /path/to/wordlist
```

### 递归扫描

通过使用 - r | --recursive 参数，dirsearch 将自动对找到的目录进行暴力破解。

```
python3 dirsearch.py -e php,html,js -u https://target -r
```

您可以使用 - R 或 --recursion-depth 设置最大递归深度

```
python3 dirsearch.py -e php,html,js -u https://target -r -R 3
```

### 线程数

线程号（-t | --threads）反映了单独的暴力破解进程的数量，每个进程将针对目标执行路径暴力破解。因此，线程数越大，目录搜索运行得越快。默认情况下，线程数为 20，但是如果您想加快进度，可以增加它。

尽管如此，速度实际上仍然不可控，因为它很大程度上取决于服务器的响应时间。作为警告，我们建议您不要将线程数过大，以免受到过多自动化请求的影响，因此应对其进行调整以适合您要扫描的系统的功能。

```
python3 dirsearch.py -e php,htm,js,bak,zip,tgz,txt -u https://target -t 30
```

### 前缀 / 后缀

•--prefixes：向所有条目添加自定义前缀

```
python3 dirsearch.py -e php -u https://target --prefixes .,admin,_,~
```

基本单词表：

```
tools<br style="box-sizing: border-box;">
```

生成的前缀：

```
.toolsadmintools_tools~tools
```

•--suffixes：向所有条目添加自定义后缀

```
python3 dirsearch.py -e php -u https://target --suffixes ~,/
```

基本单词表：

```
index.phpinternal
```

生成后缀：

```
index.php~index.php/internal~internal/
```

### 排除扩展

使用 **-X | --exclude-extensions** 与您的 exclude-extension 列表一起删除单词列表中具有给定扩展名的所有条目

```
python3 dirsearch.py -e asp,aspx,htm,js -u https://target -X php,jsp,jspx
```

基本单词表：

```
adminadmin.%EXT%index.htmlhome.phptest.jsp
```

后：

```
adminadmin.aspadmin.aspxadmin.htmadmin.jsindex.html
```

### 词表格式

支持的单词列表格式：大写，小写，大写

### 小写：

```
adminindex.htmltest
```

### 大写：

```
ADMININDEX.HTMLTEST
```

### 筛选器

使用 **-i | --include-status **和** -x | --exclude-status** 选择允许和不允许的响应状态代码

```
python3 dirsearch.py -e php,html,js -u https://target -i 200,204,400,403 -x 500,502,429
```

还支持 **--exclude-sizes**，-- **exclude-texts**，-- **exclude-regexps** 和 **--exclude-redirects** 以使用更高级的过滤器

```
python3 dirsearch.py -e php,html,js -u https://target --exclude-sizes 1B,243KBpython3 dirsearch.py -e php,html,js -u https://target --exclude-texts "403 Forbidden"python3 dirsearch.py -e php,html,js -u https://target --exclude-regexps "^Error$"
```

### 扫描子目录

### 扫描子目录

您可以从 URL 中使用 **--subdirs** 扫描子目录。

```
python3 dirsearch.py -e php,html,js -u https://target --subdirs admin/,folder/,/
```

此功能的反向版本是 **--exclude-subdirs**，它可以防止目录搜索进行强行强制执行的目录，而在执行递归扫描时，这些目录不应被强行使用。

```
python3 dirsearch.py -e php,html,js -u https://target --recursive -R 2 --exclude-subdirs "server-status/,%3f/"
```

### 代理

Dirsearch 支持 SOCKS 和 HTTP 代理，具有两个选项：代理服务器或代理服务器列表。

```
python3 dirsearch.py -e php,html,js -u https://target --proxy 127.0.0.1:8080python3 dirsearch.py -e php,html,js -u https://target --proxy socks5://10.10.0.1:8080python3 dirsearch.py -e php,html,js -u https://target --proxylist proxyservers.txt
```

### 报告

Dirsearch 允许用户将输出保存到文件中。它支持多种输出格式，例如 text 或 json，并且我们会不断更新新格式

```
python3 dirsearch.py -e php -l URLs.txt --plain-text-report report.txtpython3 dirsearch.py -e php -u https://target --json-report target.jsonpython3 dirsearch.py -e php -u https://target --simple-report target.txt
```

### 其他命令

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt -H "X-Forwarded-Host: 127.0.0.1" -f
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt -t 100 -m POST --data "user
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt --random-agent --cookie "isAdmin=1"
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt --json-report=target.json
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt --header-list rate-limit-bypasses.txt
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt --minimal 1
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt -q --stop-on-error
```

```
python3 dirsearch.py -e php,txt,zip -u https://target -w db/dicc.txt --full-url
```

```
python3 dirsearch.py -u https://target -w db/dicc.txt --no-extension
```

还有更多功能，需要自己发现哦

### 提示

• 要以每秒的请求速率运行 dirsearch，请尝试 `-t <rate> -s 1`• 是否要查找配置文件或备份？试用`--suffixes ~`和`--prefixes .`• 对于您不想强制扩展的某些端点，请`%NOFORCE%`在它们的末尾添加 • 是否只想查找文件夹 / 目录？结合`--no-extension`和`--suffixes /`！• 的组合`--cidr`，`-F`并且`-q`将降低大部分噪声 + 假阴性的用 CIDR 当暴力破解

### 支持 Docker

### 安装 Docker

```
curl -fsSL https://get.docker.com | bash
```

### 建立映像目录搜寻

创建图像

```
docker build -t “ dirsearch：v0.4.1 ” 。
```

### 使用目录搜索

用于

```
docker run -it --rm "dirsearch:v0.4.1" -u target -e php,html,js,zip
```

### 项目地址

https://github.com/suifengg/dirsearch

References

`[1]` www.certificate-transparency.org 和 crt.sh。: _http://www.certificate-transparency.org 和 crt.sh。_  
`[2]` 此处: _https://github.com/maurosoria/dirsearch/archive/master.zip_  
`[3]` Docker: _https://github.com/maurosoria/dirsearch#support-docker_  
`[4]` SecLists: _https://github.com/danielmiessler/SecLists_

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC7IHABFmuMlWQkSSzOMicicfBLfsdIjkOnDvssu6Znx4TTPsH8yZZNZ17hSbD95ww43fs5OFEppRTWg/640?wx_fmt=gif)

●[干货 | 渗透学习资料大集合（书籍、工具、技术文档、视频教程）](http://mp.weixin.qq.com/s?__biz=MzIwMzIyMjYzNA==&mid=2247490892&idx=2&sn=5820f8871f23ffc525a27e1c6ae1ae4c&chksm=96d3e649a1a46f5f88051b88fb05efd4cda4c885a4f47ac63795354cbfe5e3a93de747f3f10a&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhhSkbmEwrfDUq1W6Fpg5zBtN3oPC3DwGUCFdGJ8pj9KdcX4hicBzNsfw/640?wx_fmt=jpeg)

公众号

**点击上方，关注公众号**

如文章对你有帮助，请支持点下 “赞”“在看”