> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nZgTR3bV2fQ8WBYcxEHOxA)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2n6icNkn3AUJmRyHJLrePfibI336mu6mDBm3ZbNO9wfCs0M1GFsiaGAVmaePlJ6B7FMicyyibiaX4vtzpg/640?wx_fmt=png)

单密码登录

```
python WpCrack.py -t http://site.com/wp-login.php -u admin -p 密码
```

使用多个密码登录

```
python WpCrack.py -t http://site.com/wp-login.php -u admin --p wordlist.txt
```

关于
--

*   快速登录
    
*   HTTP 代理的使用
    
*   多线程或多处理器
    

        WpCrack 是用于强制登录 WordPress CMS Web 应用程序的工具，它是使用 Python 编程语言构建的。

https://github.com/22XploiterCrew-Team/WordPress-Brute-Force