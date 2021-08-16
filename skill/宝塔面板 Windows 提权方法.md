> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Pnco7iZo7S4SxwjzSX4fAA)

本项目整理一些宝塔特性，可以在无漏洞的情况下利用这些特性来增加提权的机会。

Table of Contents
=================

*   宝塔面板 Windows 提权方法
    

*   写数据库提权
    
*   API 提权
    
*   计划任务提权
    

*   自动化测试
    

写数据库提权
------

宝塔面板在 2008 安装的时候默认 www 用户是可以对宝塔面板的数据库有完全控制权限的：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3MhdVtYwPwjJZJuibtbAyhSI1DS7G3s4XEgQP2TS3D3lD00O9ic3pPndaFuLHkUb9TlYVNCzaUs1ibg/640?wx_fmt=png)

对于这种情况可以直接往数据库写一个面板的账号直接获取到面板权限，而在 2016 安装默认是 User 权限可读不可写

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3MhdVtYwPwjJZJuibtbAyhSzEkkHadjo77wvJpQQiaIdTUyPgibOGHRchBiaTD4iau4OMsgLJUItGRXHQ/640?wx_fmt=png)

这种情况可以从里面读取一些敏感信息，比如 mysql 的 root 密码，而一般这个配置的不会只有这个文件可读，可以使用其他方法。

盐： `[A-Za-z0-9]{12}`

密码： `md5(md5(md5(password) + '_bt.cn') + salt)`

可以直接使用`bt_panel_script.py`，脚本会自动新建一个账号。

API 提权
------

宝塔面板支持 API 操作的，token 在`C:\BtSoft\panel\config\api.json`，用这个方法提权还可以无视入口校验，比如有一个未授权访问的 redis 是 system 权限，就可以直接往这个文件覆盖 token 直接接管面板，或是利用 FileZilla(windows 面板默认 ftp 软件就是 FileZilla + 空密码) 新建一个 C 盘权限的账号，也可以去修改那个文件来提权。

API Token: `md5(string)`

api.json

```
{"open": true, "token": "API Token", "limit_addr": ["你的IP"]}
```

请求时加上 (`multipart/form-data`)：

```
request_token = md5(timestamp + token)

request_time = timestamp
```

可以直接使用`bt_panel_api.py`，脚本会自动使用计划任务运行命令，如果面板原本就有配置好 API 了，并且 IP 限制 127.0.0.1，那么就可以直接在服务器直接用脚本提权。

计划任务提权
------

基本上场景同 API 提权，可以去修改计划任务文件 (比如网站备份)，默认是在凌晨 1：30 执行，权限也是 system。

路径： `C:/BtSoft/cron/`

有些面板 API 会无法登陆，就只能利用计划任务来提权了，缺点是路径不固定，且执行时间也不固定。

自动化测试
=====

```
python3 .\bt_panel_script.py
```

使用此脚本可以全自动获取宝塔相关信息，python 可以直接用宝塔的，不用担心没环境。

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3MhdVtYwPwjJZJuibtbAyhS1VqT3Tel3zuF67doAQNKBoMic8vWpdvtibzwyN7qhMVSwWUCzPbMqDIQ/640?wx_fmt=png)

```
python3 .\bt_panel_api.py -g
```

这个脚本可以生成 api 示例，把生成的 json 替换到指定文件后就能提权。

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3MhdVtYwPwjJZJuibtbAyhSpva0AGlw7COcClJp9OibiboCd4ZCuleNzoqR4dxlKn0xHcYibQQI1uMkw/640?wx_fmt=png)

```
python3 .\bt_panel_api.py -u "http://192.168.101.5:8888/" -t "085bd64a698cf601ae472425656b2346" -c whoami
```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3MhdVtYwPwjJZJuibtbAyhSHZY7nln2m1GRP5RvnzpkURsRSc8ZpTFfkwF7f5lxn5RNsSsUGAgLPA/640?wx_fmt=png)

```
python3 .\bt_panel_log_delete.py
```

这个脚本可以自动清理面板日志。

https://github.com/Hzllaga/BT_Panel_Privilege_Escalatio

```
END



●干货|渗透学习资料大集合（书籍、工具、技术文档、视频教程）
如文章对你有帮助，请支持点下“赞”“在看”
```