> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/i4DG7x-kuO7zRLYYWR8Agw)

**前言**

去年年底看到 @madcoding 老哥博客的 “Waf 的识别与绕过” 一文中搜集了不少 WAF 拦截页面，正好我平时也有搜集 WAF 拦截页面的习惯，所以结合他的一起整理成了这么一篇文章，便于自己日后遇到 WAF 时好查询，并进行针对性的绕过测试！！！  

现如今 WAF 种类繁多，笔者搜集的可能也并不是那么齐全和准确，还望得到各位老哥的补充和修正！！！

  

**(1) D 盾**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlibIvIgcOZYh2TpwQkZB5Fca3zghL08RSwG3LhwTNyQQ9rbnr7hsmMAQ/640?wx_fmt=png)

**(2) 云锁**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Plg9bBTfbg955ibRfKm0eMRkNNmwiaomy3QibpeCRT514vY44ayhQgYGWYA/640?wx_fmt=png)

**(3) UPUPW 安全防护**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pl81Dv6Q6OxJ6swdh5vdibibXwJVUY0tFhJ5jaCqwduvTFo5TH2E78WYAw/640?wx_fmt=png)

**(4) 宝塔网站防火墙**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pl9E06m9N01tPDX7CuMlCN4hgjTjG5mFq51cUib622CJqhvXvD28VDuDw/640?wx_fmt=png)

**(5) 网防 G01**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlicyArj47Fx8YfLNutvq6uBnSczdhOPRl6SpCHbrU4dyeUU7GcAusqMA/640?wx_fmt=png)

**(6) 护卫神**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlgRCd2jgv0pxiahYOu99GR1jDJialhdq49PmMiaIglVuiacLfwiasuJ12W2Q/640?wx_fmt=png)

**(7) 网站安全狗**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlDRQicJ6PjKT4tKlk6oxiag1mmBdM9cumwg6TaIibXicfWZuDqfibNcZ2v6g/640?wx_fmt=png)

**(8) 智创防火墙**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlzwTnicWgUOlMyG7f5BdK2pl31GSEmicEc6Fat9tkle5ibbQQ8QhjcBibRw/640?wx_fmt=png)

**(9) 360 主机卫士或 360webscan**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlmwIWwib129gjXVkzGuFQnWEiayDX9pFSOsmjuevOhtBOtY8yib4CKlPew/640?wx_fmt=png)

**(10) 西数 WTS-WAF**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlqpXjuUSUjhAWdwrEchBKxVhiaPc8Gz1Q3VnJOibCTAL44wXMwa5RAic3Q/640?wx_fmt=png)

**(11) Naxsi WAF** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pl1G2XKxB1sSqxCYF1JBMWgWWicT6iczZ0ibj9zkGqRBFE9AwNhUAkVIqog/640?wx_fmt=png)

**(12) 腾讯云** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlAwhib6YWLGGuAK8VqDalp6qghBofOB6SoRm70FvLp03FrA8kG2WibUfw/640?wx_fmt=png)

**(13) 腾讯宙斯盾** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Plicu5Lfs5Xs3yrU4s6OaDkRxAPUmFqUlfmEfvD24coqldJel3j6MFSTQ/640?wx_fmt=png)

**(14) 百度云** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pljm9AYeDHGbZwWoSjCTdqPQdhE0dAh1byx4ud7RbjXWFm3ibkGQoObwQ/640?wx_fmt=png)

**(15) 华为云**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pl6ic4GIicMCmLtJeytrg5DOJDQRhYMgwF1t6oVcwuycrMU85TpNpbG3lQ/640?wx_fmt=png)

**(16) 网宿云**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlYpWEoLSsSDWkengib6UaFIOE1royzCSj25WDl134cI4dKPs16dwCGgw/640?wx_fmt=png)

**(17) 创宇盾** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlOHH4F34cmc0RCT6kKIqun2ia2AENqq6sWzMiaG1ZGvxqQibpcSnYhrCYg/640?wx_fmt=png)

**(18) 玄武盾**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlR1htqGXtU0XkwqE1Zic2nOu49gNibh0K5OgDd5bzpklqskKR04hPwzvg/640?wx_fmt=png)

**(19) 阿里云盾** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlpwDxwdUtO8ib5sTbU2zgMplibrW2n1c5Dta9jp99oYE2OlBlzL4NVclw/640?wx_fmt=png)

**(20) 360 网站卫士** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlvfN2W5C20k1xAbZ8ib5zgKkfRgXEgHjIn3cTVw5UC1MwZMfyibChSGcQ/640?wx_fmt=png)

**(21) 奇安信网站卫士** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlZSILdJBYquFgnFZWaibV7ibXO0mNbRBz0DzQ5QTMd8xUpQibibqluXgomA/640?wx_fmt=png)

**(22) 安域云 WAF**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlDRENoFeSvaFdU2udibcXcxbcZTA8BYHHTqYUIRV3dfsFY3cGQKYicMzA/640?wx_fmt=png)

**(23) 铱讯 WAF** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlVfJic6xJb9d36pU3YCZr0PQt0JUXm5PtccSmlEtC8aUnCUTwezxtxQg/640?wx_fmt=png)

**(24) 长亭 SafeLine**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pl8YHiaszfNPs1quUFrqJj823S8VtqO0HRuZloiahYEuZv2vNW6xA2ib3qg/640?wx_fmt=png)

**(25) 安恒明御 WAF** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlJofBfrTfTf3EqscKoDcOdeK2LE5uNRGkgaw7oia9g0icTZcGWwozx1UQ/640?wx_fmt=png)

**(26) F5 BIG-IP**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlAyBk2If7QMMpiaA1GdVUb0K7n8mrEOvwsibCIyyBF8tGw2WBBuJfOLkw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlkmTKPbEl7QC4nTLicZwE2d9g7qwq8ibbxB3CiczbqrEVtgdEAzmKaZpww/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlC9b8ibX9aWXplS0T6ItbmPibE070YUZdQjuvlp82nFDzxY64JJmtP37g/640?wx_fmt=png)

**(27) Mod_Security**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlKibEBgUMkKB3FvEJZG9TLs1q9Y0O3sNic51uoGqQacmQaSHkhXwC8qgQ/640?wx_fmt=png)

**(28) OpenRASP**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlQMia69FTZ2mHVFcRNwBeyL6ic65ibabEMlhHqG34CYiaMKXPqEW9Y45N5g/640?wx_fmt=png)

**(29) dotDefender** 

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33PlljR981yxlPs1S8icxBt3EamibC4eocvJ7BTEwApWibegibckYtN9EVG0PA/640?wx_fmt=png)

**(30) 未知云 WAF**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdOBYLCxjaJTmkwFw7o33Pl4sKhPlMvvHOTuwIkatvx4uzWOdd9eaNK8ialQ0WIPLgRWcL2EZKLv9A/640?wx_fmt=png)

作者：3had0w，文章来源潇湘信安

****扫描关注乌雲安全****  

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavz34wLFhdnrWgsQZPkEyKged4nfofK5RI5s6ibiaho43F432YZT9cU9e79aOCgoNStjmiaL7p29S5wdg/640?wx_fmt=jpeg)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**