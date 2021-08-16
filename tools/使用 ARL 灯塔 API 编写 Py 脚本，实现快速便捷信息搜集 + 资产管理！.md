> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5q__NZflG5Ome22UkpGSoA)

**演示视频**

 **前言**  

MD！写的时候忘记记录了... 现在你们看到的是我根据我的记忆来写的文章...

    因为最近项目有点多，信息搜集耗时有点慢，所以想到之前用到一个工具 “ARL 灯塔”！但是每次都要登陆 WEB 端点点点就很费时间，所以

**开发**

```
from re import S
from cryptography.hazmat.primitives.asymmetric.ec import BrainpoolP256R1
import requests
import readline
import json
from prettytable import PrettyTable
import urllib3
from os import system, name
import urllib.request
```

    json 库就是拿来解析返回结果的，prettytable 是拿来美观输出结果的，urllib3 和 requests 就是拿来做请求的。

**其他的我也忘记都是干啥的了～**

```
apikey = "ff44256c-fa2b-483f-9956-b0a84e153ade"
```

    灯塔的 API 请求需要一个 key，这个 key 是在 “**config-docker.yaml**” 文件内配置的，然后我们把我们配置的 key 赋值给一个变量就好了。

```
def task():
    table = PrettyTable(['任务名称', '目标地址', '当前状态', '开始时间', '结束时间', '任务ID'])
    headers = {
    'accept': 'application/json',
    'Token': apikey
    }
    ceshi = requests.get("https://IP:5003/api/task/", headers=headers, verify=False)
    good = ceshi.text
    res = json.loads(good.encode('latin-1').decode('unicode_escape'))
    for i in res['items']:
        table.add_row([i['name'],i['target'],i['status'],i['start_time'],i['end_time'],i['_id']])
    else:
        print(table)
    table.clear_rows()
```

    key 配置好之后我们定义一个任务列表获取功能，这里可以看到我们的任务列表，效果就是下图这样子

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTqnauXiaym5TYDjYo62P7OmXvmLaSLRRAO7ZCxvTfK21KS4MToUmzfRw/640?wx_fmt=jpeg)

```
def add(name, target):
    headers = {
    'Content-type': 'application/json',
    'Token': apikey,
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
    }
    data = {
        "name": name,
        "task_tag": "task",
        "target": target,
        "policy_id": "策略ID",
        "result_set_id": "策略ID"}
    jsondata = json.dumps(data, separators=(',', ':'), ensure_ascii=False)
    ceshi = requests.post("https://IP:5003/api/task/policy/", headers=headers, data=jsondata, verify=False)
    good = ceshi.text
    res = json.loads(good.encode('latin-1').decode('unicode_escape'))
    if res['message'] == 'success':
        print(res['message'] + '  \033[1;31m下发任务成功！\033[0m')
    else:
        print('\033[1;32m下发任务失败！\033[0m')
```

    灯塔的任务下发接口我感觉不如用策略下发功能，所以我这里用的是策略下发功能，策略下发需要获取**策略 ID**！

**首先我们需要新建策略：**

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTI72KzfWYCYmJ2ZeyCk9BKiaiahn4ccDelicAbfdFo05FOT1RnHqWia1OmQ/640?wx_fmt=jpeg)

然后我们访问：**https://IP:5003/api/doc**

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTsPRNQs7ziawE1Q0y1VMak2JE9mYMYSOdpr4YicGmpcffFdPcMnwQ2acQ/640?wx_fmt=jpeg)

点击右侧绿色按钮填入自己的 KEY 值后点击绿色按钮即可

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTdf4uTSmiaBjOicEshLeUXUG5qjWLy1xMic9w54Riaz13s1bPD13CTswz6A/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTkdRcwCEia8tRzKcJAluHhj4WyZNiblwoNK8Lw3cCib47OzBh8KysjiaeIQ/640?wx_fmt=jpeg)

然后拉到最下面可以点开 “**策略信息** “一栏，然后选择 “**策略信息查询** “一栏，最后点击右侧灰色按钮

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTvVmYRt5aEv5mhjrK5Ssr2SKeZ8UuZZEQQUibicb93Eoz6ejibibAWx0FQA/640?wx_fmt=jpeg)**不需要填写任何信息，直接点击蓝色按钮**

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTCug1oufOcv4ud3IK4X232nhO5B96t22BpYtcMk5OmicJTkdiaiaCXuYXQ/640?wx_fmt=jpeg)往下拉可以看到返回结果，"**_id**" 参数为我们要获取的**策略 ID**

![](https://mmbiz.qpic.cn/mmbiz_jpg/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaT2GMf0iaYWzeC0465SBibGt4olVEsAIl8bWetMBecVx92aYgFcTde03Tg/640?wx_fmt=jpeg)

    整体功能其实就是这样完成的，其他功能自行摸索吧，剩下的就是把调用函数实现功能就好了，自行看下面的代码吧..

```
xunhuan = 1
while xunhuan == 1:
    a = input("\033[1;31m>> \033[0m")
    if a == 'ls':
        task()
    if a == 'addfile':
        name = input('\033[1;34m[+] >> 任务名称：\033[0m')
        file = input('\033[1;34m[+] >> 目标文件：\033[0m')
        with open(file, 'r', encoding='utf-8') as file1:
            for readFile in file1: #逐行读取
                add(name.encode("utf-8","ignore").decode("latin-1"), readFile)
    if a == 'add':
        print('\033[1;31m[+] 任务目标英文逗号分割！例：1.2.3.4,4.3.2.1\033[0m')
        print('\033[1;32m--------------------------------------------------\033[0m')
        name = input('\033[1;34m[+] >> 任务名称：\033[0m')
        target = input('\033[1;34m[+] >> 任务目标：\033[0m')
        add(name.encode("utf-8","ignore").decode("latin-1"), target)
    if a == 'c':
        clear()
    if a == 'e':
        break
```

**结尾**

    本来想做成类似 MSF 那样子的**等待命令输入的交互式**，但是没有找到相关库，也不知道怎么写，所以就写了个**死循环**然后**输入的内容判断**来实现的那种效果。

开源地址：本公众号 云剑侠心或者回复 “**ARL 脚本**” 即可获取下载链接！  

公众号

替换脚本内 url 地址的时候要像下图这样替换掉 IP，否则可能会报错！

![](https://mmbiz.qpic.cn/mmbiz_png/CwmXBonxXzcibWU4p2D9Dib3CtbCKxFZiaTXOGWpF5cicCsJv4WqMANvz3rWpZs0Aict3jfAogHECoUNlBRNHCia6Ljw/640?wx_fmt=png)

    **addfile** 其实就是通过读取指定文件然后逐行读取循环下发任务给 **add 函数**。

各位大佬如果有好的想法可以私信我微信来交流嗷：**backxyh**