> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMzIwNDYyNg==&mid=2650624995&idx=1&sn=14b4f560e6d38bccf29fde3dfc6dac79&chksm=8337409eb440c98887b265d8afb9bb3dca9f0456d9b2880f5bcf2f22385c57c1d168e0b009b1&mpshare=1&&srcid=&sharer_sharetime=1574872612322&sharer_shareid=cb64def495bb2bef15b1a9a543cdd734&from=timeline&scene=2&subscene=1&clicktime=1574904050&enterid=1574904050#rd)

ssh 是 Uinx 派系系统中通用的管理接口协议，默认端口 22。这个协议稳定而且通用性强，过防火墙、过检测、过中间人嗅探那是一等一的尖端，而且，机构内部基本对此协议通行无阻，实乃红队（攻方）穿墙越网之居家必备。

用 ssh 命令当做流量和端口转发完全可以利用上面的优势，就是密令复杂了一点。下面介绍的这个工具 Sshtunnel，是 Python 的一个安装包，它不仅能直接以命令行方式实现流量转发，还有更强大的编程功能。

![](https://mmbiz.qpic.cn/mmbiz_png/VT33z2qICCybUToAemKLfYd06BjAFnKJ3Qp42oX3XvZKLkbVHKXsBicOjDR6vYBmfzoLugTIRqtUEd9LpovEdTg/640?wx_fmt=png)

  

**A. 安装:**

sshtunnel 是 PyPI 维护的包, 也就是说，简单地用 python 包安装命令就能安装：

```
pip install sshtunnel
```

或者

```
easy_install sshtunnel
```

或者

```
conda install -c conda-forge sshtunnel
```

源码安装，先从这里（https://github.com/pahaz/sshtunnel）下回来，然后运行：

```
python setup.py install
```

**B. 使用场景：**

使用 sshtunnel 的一个经典场景，就如下面图形描述的那样。用户可能需要访问远程服务器（例如 8080 端口），但是你只能访问这台远程服务器的 SSH 端口（通常是 22 端口）

![](https://mmbiz.qpic.cn/mmbiz_png/VT33z2qICCybUToAemKLfYd06BjAFnKJ4FlfnDNqDqMgsBLjsVYuo5CplIHdZCY2gLk5wab3y1NQ6lMbwO21YA/640?wx_fmt=png)

图 1：如何通过 ssh 绕过防火墙去访问一个不能直接访问的服务

如果能操纵 SSH 服务器，还可以访问到本地客户端看不到的私有的服务器（远程服务器的角度）

![](https://mmbiz.qpic.cn/mmbiz_png/VT33z2qICCybUToAemKLfYd06BjAFnKJhAmCU5KDrH9YVsJfva2KFqEpEAZyySUqqdFhgkhI6pguVG50icPqfCQ/640?wx_fmt=png)

图 2：如果通过 SSH 隧道访问私有服务器

**C. 用例：**

API 不仅允许初始化渠道后直接启动，还允许使用 “with” 语境，这会贯穿从隧道启动到关闭的整个过程。

**案例 1**：

上面图 1 的例子，假如远程服务器的地址是:"pahaz.urfuclub.ru", 使用密码认证，然后随机指定本地绑定的端口。

```
from sshtunnel import SSHTunnelForwarder

server = SSHTunnelForwarder(
    'pahaz.urfuclub.ru',
    ssh_user,
    ssh_password="secret",
    remote_bind_address=('127.0.0.1', 8080)
)

server.start()

print(server.local_bind_port)  # show assigned local port
# work with `SECRET SERVICE` through `server.local_bind_port`.

server.stop()
```

**案例 2**:

上面图 2 的例子，不能直接访问的私有服务端口转发案例，假如只能使用证书访问，远程 SSH 服务器只有 443 端口没被左边的防火墙拦截。

```
import paramiko
import sshtunnel

with sshtunnel.open_tunnel(
    (REMOTE_SERVER_IP, 443),
    ssh_user,
    ssh_pkey="/var/ssh/rsa_key",
    ssh_private_key_password="secret",
    remote_bind_address=(PRIVATE_SERVER_IP, 22),
    local_bind_address=('0.0.0.0', 10022)
) as tunnel:
    client = paramiko.SSHClient()
    client.load_system_host_keys()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    client.connect('127.0.0.1', 10022)
    # do some operations with client session
    client.close()

print('FINISH!')
```

**案例 3**：

Mysql 3306 端口转发：

```
from sshtunnel import open_tunnel
from time import sleep

with open_tunnel(
    ('localhost', 2222),
    ssh_user,
    ssh_password="vagrant",
    remote_bind_address=('127.0.0.1', 3306)
) as server:

    print(server.local_bind_port)
    while True:
        # press Ctrl-C for stopping
        sleep(1)

print('FINISH!')
```

或者简单的一条命令：

```
(bash)$ python -m sshtunnel -U vagrant -P vagrant -L :3306 -R 127.0.0.1:3306 -p 2222 localhost
```

**案例 4**：

启动一个跳跨两个隧道的 ssh session。SSH 传输和隧道都是以后台形式运行，它不会在连接关闭时退出进程。

```
import sshtunnel
from paramiko import SSHClient


with sshtunnel.open_tunnel(
    ssh_address_or_host=('GW1_ip', 20022),
    remote_bind_address=('GW2_ip', 22),
    block_on_close=False
) as tunnel1:
    print('Connection to tunnel1 (GW1_ip:GW1_port) OK...')
    with sshtunnel.open_tunnel(
        ssh_address_or_host=('localhost', tunnel1.local_bind_port),
        remote_bind_address=('target_ip', 22),
        ssh_username='GW2_user',
        ssh_password='GW2_pwd',
        block_on_close=False
    ) as tunnel2:
        print('Connection to tunnel2 (GW2_ip:GW2_port) OK...')
        with SSHClient() as ssh:
            ssh.connect('localhost',
                port=tunnel2.local_bind_port,
                username='target_user',
                password='target_pwd',
            )
            ssh.exec_command(...)
```

命令行使用帮助：

```
$ sshtunnel --help
usage: sshtunnel [-h] [-U SSH_USERNAME] [-p SSH_PORT] [-P SSH_PASSWORD] -R
                 IP:PORT [IP:PORT ...] [-L [IP:PORT [IP:PORT ...]]]
                 [-k SSH_HOST_KEY] [-K KEY_FILE] [-S KEY_PASSWORD] [-t] [-v]
                 [-V] [-x IP:PORT] [-c SSH_CONFIG_FILE] [-z] [-n] [-d [FOLDER [FOLDER ...]]]
                 ssh_address

Pure python ssh tunnel utils
Version 0.1.5

positional arguments:
  ssh_address           SSH server IP address (GW for SSH tunnels)
                        set with "-- ssh_address" if immediately after -R or -L

optional arguments:
  -h, --help            show this help message and exit
  -U SSH_USERNAME, --username SSH_USERNAME
                        SSH server account username
  -p SSH_PORT, --server_port SSH_PORT
                        SS   H server TCP port (default: 22)
  -P SSH_PASSWORD, --password SSH_PASSWORD
                        SSH server account password
  -R IP:PORT [IP:PORT ...], --remote_bind_address IP:PORT [IP:PORT ...]
                        Remote bind address sequence: ip_1:port_1 ip_2:port_2 ... ip_n:port_n
                        Equivalent to ssh -Lxxxx:IP_ADDRESS:PORT
                        If port is omitted, defaults to 22.
                        Example: -R 10.10.10.10: 10.10.10.10:5900
  -L [IP:PORT [IP:PORT ...]], --local_bind_address [IP:PORT [IP:PORT ...]]
                        Local bind address sequence: ip_1:port_1 ip_2:port_2 ... ip_n:port_n
                        Elements may also be valid UNIX socket domains:
                        /tmp/foo.sock /tmp/bar.sock ... /tmp/baz.sock
                        Equivalent to ssh -LPORT:xxxxxxxxx:xxxx,    being the local IP address optional.
                        By default it will listen in all interfaces (0.0.0.0) and choose a random port.
                        Example: -L :40000
  -k SSH_HOST_KEY, --ssh_host_key SSH_HOST_KEY
                        Gateway's host key
  -K KEY_FILE, --private_key_file KEY_FILE
                        RSA/DSS/ECDSA private key file
  -S KEY_PASSWORD, --private_key_password KEY_PASSWORD
                        RSA/DSS/ECDSA private key password
  -t, --threaded        Allow concurrent connections to each tunnel
  -v, --verbose         Increase output verbosity (default: ERROR)
  -V, --version         Show version number and quit
  -x IP:PORT, --proxy IP:PORT
                        IP and port of SSH proxy to destination
  -c SSH_CONFIG_FILE, --config SSH_CONFIG_FILE
                        SSH configuration file, defaults to ~/.ssh/config
  -z, --compress        Request server for c   ompression over SSH transport
  -n, --noagent         Disable looking for keys from an SSH agent
  -d [FOLDER [FOLDER ...]], --host_pkey_directories [FOLDER [FOLDER ...]]
                        List of directories where SSH pkeys (in the format `id_*`) may be found
```

原文：https://www.zdnet.com/article/microsoft-russian-hackers-are-targeting-sporting-organizations-ahead-of-tokyo-olympics/

HackerHub 发布 | 转载请注明出处  

[.>](http://mp.weixin.qq.com/s?__biz=MzAwMzIwNDYyNg==&mid=2650624864&idx=1&sn=91b3d5a9dc97a3fb3b0b1581672fdd09&chksm=8337401db440c90b9458381aea746aac7df5458147d95c2b974d9e4c9da4c21a31abf3505a06&scene=21#wechat_redirect) [勒索软件绕过安全监测的那些套路](http://mp.weixin.qq.com/s?__biz=MzAwMzIwNDYyNg==&mid=2650624949&idx=1&sn=f25c682d06a9ecfa7092fe822ddabbb8&chksm=833740c8b440c9de6aeace81f6cb6ee64a944e52b2bdd80092b427af8e50e19bcf48997ef22c&scene=21#wechat_redirect)

.> [手把手，远程桌面客户端提取明文账号密码](http://mp.weixin.qq.com/s?__biz=MzAwMzIwNDYyNg==&mid=2650624988&idx=1&sn=85338795727eac0ff2a7ebdce22a23c5&chksm=833740a1b440c9b7ec42ebf36f99802c4e885ae09185de90fd2503535170c1a4cad438765a1c&scene=21#wechat_redirect)

.> [一种通用的 Docker 提权方式](http://mp.weixin.qq.com/s?__biz=MzAwMzIwNDYyNg==&mid=2650624967&idx=1&sn=7ede63107c40d3591defa693bf739faa&chksm=833740bab440c9ac6aae28c5077a1718d8be458a16918fd8cea269ef61115bcaec6b7b0cc19b&scene=21#wechat_redirect)

**我在看，你呢？**![](https://mmbiz.qpic.cn/mmbiz_gif/kw2nrMk65sdViahJApicmvFAv8pTOsGaMte4A4GHkOyRUPDCYsNGA0jso4dkb169ib9Hiaq8RwAUjHn4CWnW9J81jA/640?wx_fmt=gif)