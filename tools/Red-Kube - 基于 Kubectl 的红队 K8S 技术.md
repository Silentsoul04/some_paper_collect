> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3Y3u3Q7vTfIuO_UXbN-rHg)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1zT1QkibZ7yibJoADwVUvuJuicx74QzHF6IxtSurOdkYd26VwplW7cGf7Kq6EpICtvU4tdaGUqNxcWw/640?wx_fmt=png)

        Red Kube 是 kubectl 命令的集合，编写这些命令是为了从攻击者的角度评估 Kubernetes 集群的安全状态。

        这些命令对于数据收集和信息公开是被动的，对于执行影响集群的实际操作是主动的。

        这些命令已映射到 “MITRE ATT＆CK” 战术，以帮助您了解我们最大的差距所在，并优先考虑我们的发现。

        当前版本包装有一个 python 编排模块，可根据不同的场景或策略在一次运行中运行多个命令。

        请谨慎使用，因为某些命令处于活动状态并正在积极部署新容器或更改基于角色的访问控制配置。

警告：您不应该在非您拥有的 Kubernetes 集群上使用 red-kube 命令！

python3 要求

```
pip3 install -r requirements.txt
```

kubectl（Ubuntu / Debian）

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

```
cat << EOF > /etc/yum.repos.d/kubernetes.repo 
[kubernetes] 
name = Kubernetes 
baseurl = https：//packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 
enabled = 1 
gpgcheck = 1 
repo_gpgcheck = 1 
gpgkey = https：//packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key。gpg 
EOF 

yum install -y kubectl
```

kubectl（基于 Red Hat）

```
usage: python3 main.py [-h] [--mode active/passive/all] [--tactic TACTIC_NAME] [--show_tactics] [--cleanup]

required arguments:
--mode            run kubectl commands which are active / passive / all modes
--tactic          choose tactic

other arguments:
-h --help         show this help message and exit
--show_tactics    show all tactics
```