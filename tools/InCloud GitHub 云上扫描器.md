> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IntTPw4VpgaVzbZd1BZ8IQ)

###                                                       

**描述  
**

使用 GitHub 云扫描，实现 ip 隐藏、防溯源、云上自动化信息收集。

**工具定位  
**

  
运行于 GitHub Actions 的仓库中自动化、自定义和执行软件开发工作流程，可以自己根据喜好定制功能，InCloud 已经为您定制好了八种针对网段和域名的不同场景的信息收集与漏洞扫描流。

功能：

*   PortScan-AllPort 对单 IP 文件列表进行全端口扫描，输出可用 Web 服务标题。
    
      
    
*   PortScan-AllPort-Xray-Dirscan 对单 IP 文件列表进行全端口扫描，输出可用 Web 服务标题，对 Web 服务进行 Xray 爬虫爬取与漏洞扫描，对 Web 服务进行 Ffuf 目录递归扫描。
    
      
    
*   PortScan-Top1000 对单 C 段 IP 列表进行 Top1000 端口扫描，输出可用 Web 服务标题。
    
      
    
*   PortScan-Top1000-Xray 对单 C 段 IP 列表进行 Top1000 端口扫描，输出可用 Web 服务标题，对 Web 服务进行 Xray 爬虫爬取与漏洞扫描。
    
      
    
*   PortScan-Top1000-Dirscan 对单 C 段 IP 列表进行 Top1000 端口扫描，输出可用 Web 服务标题，对 Web 服务进行 Ffuf 目录递归扫描。
    
      
    
*   SubDomain-Portscan-Vulnscan 对域名进行子域名枚举与接口查询，对查询的子域名进行 Top1000 端口扫描，输出可用 Web 服务标题，对 Web 服务进行 Nuclei 漏洞扫描。
    
      
    
*   SubDomain-Portscan-Xray 对域名进行子域名枚举与接口查询，对查询的子域名进行 Top1000 端口扫描，输出可用 Web 服务标题，对 Web 服务进行 Xray 爬虫爬取与漏洞扫描。
    
      
    
*   SubDomain-Portscan-Dirscan 对域名进行子域名枚举与接口查询，对查询的子域名进行 Top1000 端口扫描，输出可用 Web 服务标题，对 Web 服务进行 Ffuf 目录递归扫描。
    

使用方法：

*   1. 将项目 fork 到自己的 github.
    
*   2. 修改流程文件（.github/workflows/incloud.yaml）里的 git config --local user.email 与 git config --global user.name 改成自己的邮箱与自己的 ID（用于报告输出）
    
*   3. 修改 input 目录的扫描目标，使用 action 标签进行在线编译。
    
    4.GitHub 提供六小时的容器使用时长，扫描结束后，扫描结果会自动上传到自己 fork 的 output 文件夹下。
    
      
    
      
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/yYePiaZj2cHibVicof1FC64sKWdia1IXJxmiaLN2mNVXHKK4COLicC9xux9gqOKibAHRZXZlxLBmHtwmaV3xdtuw5GMzQ/640?wx_fmt=png)
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/yYePiaZj2cHibVicof1FC64sKWdia1IXJxmiaAtNgibJg2fvdfsiboOaXE0eYNpR15nMUnDfzlSJicUwKP5ep8JWJjNQ0w/640?wx_fmt=png)
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/yYePiaZj2cHibVicof1FC64sKWdia1IXJxmia0OsRIwzweic6RkhohlYbD4403iaJzWDOkwcuFKfQ9Sl2oI1yvy85NDibQ/640?wx_fmt=png)
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/yYePiaZj2cHibVicof1FC64sKWdia1IXJxmiajvKC4nXCiaFcv6RTUGutE1ER7a2xhtlllBMpPL6PC8tX8dib3I8diawGw/640?wx_fmt=png)
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/yYePiaZj2cHibVicof1FC64sKWdia1IXJxmiaEtibjpFuWegds2PAWEr96FY4lOkibyEFIAFNwHficOEKyr8vMWaWu0kUQ/640?wx_fmt=png)
    
      
    
    **项目地址：  
    **
    
    **github.com/inbug-team/InCloud**
    
      
    
      
    
      
    
    公众号