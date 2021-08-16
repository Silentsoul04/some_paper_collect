> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LFaLCv--yUGHXM8IqaHISg)

        可从 Github 个人资料中获取信息并查找提交时泄漏的 GitHub 用户的电子邮件地址

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1dPafGtqEtOfItNcLLNIB7fxARrO7xdLAR6PkmAtl8EI0PahDibDtRWNpUYJNT9xn6wU2lRUsNw5Q/640?wx_fmt=png)

        GitHub 使用与 GitHub 帐户关联的电子邮件地址将提交和其他活动链接到 GitHub 配置文件。当用户向公共回购提交承诺时，他们的电子邮件地址通常会在提交中发布，并且可以公开访问（如果您知道在哪里查找）。

        GitHub 提供了有关如何防止这种情况发生的一些说明，但似乎大多数 GitHub 用户都不知道或不在乎其电子邮件地址可能被公开。

        查找 GitHub 用户的电子邮件地址通常就像通过 GitHub API 查看他们的近期事件一样简单。

安装：

```
git clone https://github.com/GONZOsint/gitrecon.git
cd gitrecon/
python3 -m pip install -r requirements.txt
```

用法：

```
usage: gitrecon.py [-h] -s {github,gitlab} [-a] [-o] username

positional arguments:
  username

optional arguments:
  -h, --help          show this help message and exit
  -s {github,gitlab}  sites selection
  -a, --avatar        download avatar pic
  -o, --output        save output
```

结果保存在`results/<username>/`路径中。

可获取泄露信息：

   
**个人资料信息**  

*   用户名
    
*   姓名
    
*   用户身份
    
*   头像网址
    
*   电子邮件
    
*   地点
    
*   生化
    
*   公司
    
*   博客
    
*   墓碑编号
    
*   Twitter 用户名
    

项目地址：

https://github.com/GONZOsint/gitrecon