> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247484204&idx=1&sn=c8a5afff66893bec4adb3aa475d6f594&chksm=eaad8311ddda0a074a829106ba4e61b41ea9a61e3988aa235e569b69cf9e3c44e2f8d31a1248&scene=21#wechat_redirect)

**目录**

  

Docker 容器和 KVM 虚拟化  

Docker 的安装和使用

基于 Docker 的漏洞复现环境 Vulhub 的使用 

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HzZrTT4UZAcukC3cKqetSHpGJABL8ZCM8yibLyNpvY2Zia3IAY3P6yE9A/640?wx_fmt=gif)

Docker 容器和 KVM 虚拟化

    Docker 容器是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。Docker 容器是一种轻量级、可移植、自包可以在含的软件打包技术，使应用程序几乎任何地方以相同的方式运行。开发人员在自己笔记本上创建并测试好的容器，无需任何修改就能够在生产系统的虚拟机、物理服务器或公有云主机上运行。容器是完全使用沙箱机制，相互之间不会有任何接口，几乎没有性能开销，可以很容易地在机器和数据中心中运行。最重要的是，他们不依赖于任何语言、框架包括系统。简单的说，容器就是在隔离环境运行的一个进程，如果进程停止，容器就会销毁。隔离的环境拥有自己的系统文件，IP 地址，主机名等。

**Docker 技术介绍**：Docker 是通过内核虚拟化技术（namespaces 及 cgroups cpu、内存、磁盘 io 等）来提供容器的资源隔离与安全保障等。由于 Docker 通过操作系统层的虚拟化实现隔离，所以 Docker 容器在运行时，不需要类似虚拟机（VM）额外的操作系统开销，提高资源利用率。

**Linux 容器技术，容器虚拟化和 kvm 虚拟化的区别：**

· 容器：共用宿主机内核，运行服务，损耗少，启动快，性能高

· 容器虚拟化：不需要硬件的支持。不需要模拟硬件，共用宿主机的内核，启动时间秒级 (没有开机启动流程)

· kvm 虚拟化：需要硬件的支持，需要模拟硬件，可以运行不同的操作系统，启动时间分钟级 (开机启动流程)

**Docker 和 KVM 虚拟化的优点**

· Docker 解决了软件和操作系统环境之间的依赖，能够让独立服务或应用程序在不同的环境中，得到相同的运行结果。docker 镜像有自己的文件系统。

· Kvm 解决了硬件和操作系统之间的依赖，Kvm 独立的虚拟磁盘，xml 配置文件。

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

Docker 的安装和使用

**docker 的安装**

```
curl -s https://get.docker.com/ | sh   #一键安装Docker，root权限运行。
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPqiab05kgUnhEfBzsRuz5XicgztcIG8IyE47lvV8XZ6lTACGbVp842wvA/640?wx_fmt=png)

**查看 Docker 版本** 

```
docker version
```

 **docker 服务的启动与停止**

```
systemctl start docker              #启动

systemctl stop docker               #关闭docker

systemctl restart  docker           #重启docker服务

systemctl daemon-reload             #守护进程重启
```

**docker 镜像的管理**  

```
docker images         #查看本地镜像

docker images -a      #查看所有的镜像

docker images php     #查看仓库名为php的镜像

docker rmi -f 镜像ID     #强制删除镜像

docker rmi -f 镜像名A:tag 镜像名B:tag    #删除多个镜像

docker rmi -f $(docker images -aq)      #删除全部镜像

docker save          #导出镜像      例如：docker image save centos > docker-centos7.4.tar.gz

docker load          #导入镜像       例如：docker image load -i docker-centos7.4.tar.gz

docker search xx     #查找相关镜像   例如：docker search redis

docker search -s 30 redis     #查找start大于30的redis镜像

docker pull  name:标签    #从查找的镜像中下载下来,标签默认是latest  例如：docker pull  redis 等价于 docker pull redis:latest
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPktYl8Im5icr3BrJErXRWLK0CJH85sLab4lUPPzzuRPhXibXOnEeU15fg/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPZtjVb4Y12XjHC3q6Mtgh20Q0TbPyf8o6ribRaqEVuYYVgGLHia2JmxKw/640?wx_fmt=png)

 **docker 容器的启动、停止、查看和删除**

```
docker run  -d -P --name xxx REPOSITORY:TAG                  #根据镜像启动容器

    -d:让容器在后台运行

    -P:将容器内部使用的网络端口映射到我们使用的主机上

    -p:自定义端口映射，如 -p 8002:80，意思就是将容器的80端口映射到宿主机的8002端口

    --name:该参数可选，指定容器的名字

docker ps                                   #查看运行中的容器

docker ps –a                                #查看所有的容器

docker start    容器ID                      #启动容器

docker stop     容器ID                      #停止容器

docker restart  容器ID                      #重启容器

docker rm   容器ID                          #删除容器，删除容器前需停止该容器

docker rm  `docker ps -aq`                  #删除全部容器
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPAIrOb8iccqvMMibzgzBagA2Ta4aucMicibtNPNcaRuF9SXViaC0gdFEa8uw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPecmNjOv5tq50OFCeZk15soiaDf5cYprvk943jZaEPfiaQZ8nwiaOQIV1Q/640?wx_fmt=png)

当启动容器后， 这里会有一个端口映射，此时我们访问宿主机的 9001 端口就行了

 ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPRdOmLGfra0KPgw3wibQdLUvhU9C0Nu8hXnQicFVtkSfj49agRfkZiaWvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPo8VBHXBwrAHaCL4T4iak5Zpy5d7VLsxdCmvnuUpHxs6EOZ2Q0ZcVz2A/640?wx_fmt=png)  

**进入 docker 容器进行管理** 

```
docker exec -it  容器id或容器名字 /bin/bash
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPq0CIDUxXpgNtdY4BQUV67b0HHMPIe4TDDdWDMk4djXCaDRcoictBJTg/640?wx_fmt=png)

**导入导出容器**

```
docker export 容器ID > /opt/test.tar      #导出当前容器镜像到/opt/test.tar

docker import /opt/test.tar              #导入/opt/test.tar到容器
```

**查看 WEB 应用程序日志**

```
docker logs -f 容器ID      #可以查看容器内部的标准输出。
```

**查看容器的进程**

```
docker top 容器名
```

**配置 docker 镜像加速**

docker 默认是从国外 search 和 pull，所以这里我们需要添加一个国内的镜像地址

```
#打开该文件

vim /etc/docker/daemon.json   #然后加入下面这些内容

{"registry-mirrors": ["https://registry.docker-cn.com"]}
```

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

基于 Docker 的漏洞复现环境 Vulhub 的使用

**基于 Docker 的漏洞复现环境 Vulhub 的使用** 
---------------------------------

vulhub 的地址：https://vulhub.org

Vulhub 是一个基于 docker 和 docker-compose 的漏洞环境集合，进入对应目录并执行一条语句即可启动一个全新的漏洞环境。

关于如何安装 Docker 和 Docker-compose 就不再赘述。直接启动对应靶机的容器。

启动 Docker：systemctl  start  docker

进入对应的靶机目录，这里我选择 weblogic 的 CVE-2017-10271 漏洞，直接一键启动：docker-compose up -d

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPTToteSrgcfC9vHkUIIM1TeSibqgNZYounVfwAtN8ia03ISyMBJzjmHBA/640?wx_fmt=png)

在漏洞复现完成后，还是在漏洞的目录下移除环境，命令：docker-compose  down

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jP6f8AnImZxQxevEgRHXv1TFx0CvKv16K3XrFuPL6x6r9ksfagO6TBqg/640?wx_fmt=png) 

参考文章：Docker 容器的安装与使用 

                 Docker 教程 | 菜鸟教程

                         

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640)

来源：谢公子的博客

责编：梁粉

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640)

由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)