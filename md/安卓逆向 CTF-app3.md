> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TmENLXT1iUcHfzQN78talw)

![](https://mmbiz.qpic.cn/mmbiz_gif/kicB09lvgibnnRjv0AAqQxyBODIttZXnQqcTPoF4Pt8tJmnia4CHaYUS3zqicFfKZTWibXTAew2ibFHDjy5Pf8nDnVEQ/640?wx_fmt=gif)

点击上方「蓝字」关注我们

![](https://mmbiz.qpic.cn/mmbiz_png/kLQoJJzjYaicxneNzbOg7ynx3TfnIwmNTpJQ7orkaUNrJIV4u7PNdSJ25Mtn6XdRQTamLDDicHnYfdic2bsiaNQjCw/640?wx_fmt=png)

这道题比较综合，不是纯粹的算法题。

拿到题目，看到后缀是 ab , 直接修改后缀 apk 或 zip ，尝试打开失败。百度搜索一下怎么打开这个后缀文件。

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6wibpRQnlFPLyO69a9plJmYmaGS1ZzLZ7al2tNjqckfB1yGRb8tVrry5bVwSPqw1wdIAians52BG97yg/640?wx_fmt=png)

需要特定的工具打开，在找找相关的资料浅谈安卓系统备份文件 ab 格式解析, 找到关于 ab 文件的介绍和打开工具。如果不想了解可以直接下载 abe。

命令：

![](https://mmbiz.qpic.cn/mmbiz_gif/44UT6cJicBVUGUwbPDzvHtk8BGscpG3ucFWttjjHAn9nKuuIgYFBUTnnQHd6rCFEtTN7YAhIZoEGwSdKaSLqNlA/640?wx_fmt=gif)

java -jar abe.jar unpack 399649a0e89b46309dd5ae78ff96917a.ab 399649a0e89b46309dd5ae78ff96917a.tar

![](https://mmbiz.qpic.cn/mmbiz_png/8NicMvXribe7uMvSuOzNiaduO31HtjchjrUcB2HicpDUBet2J3rTz8EjbKaRq2f8zEGWnV8x1UoNQBf8WLXZISpNIQ/640?wx_fmt=png)

解开后大致浏览一下目录，a 目录下是 apk 文件，db 目录下是个数据库文件，还有 Encryto.db 那么这里很大可能就是要查看加密的 db 文件，

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6wibpRQnlFPLyO69a9plJmYmaJJbBzBH33X18MmzCDpgicRYMBpHmD7XIlibYfRXaib81R1EvsDRpEiaUFw/640?wx_fmt=png)

使用 jadx-gui 反编译源码，查看逻辑。

类 a: 创建一个 table，看到第三个字段存在 flag 字样，那么印证了刚刚的猜想。

类 AnotherActivity : 这个是安装 apk 后第一个界面，没有什么用。

类 MainActivity ：可以获取加密的数据库类型和版本 ，a() 方法向数据库中插入了一条数据。

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6wibpRQnlFPLyO69a9plJmYmaiciaTgnhFpMsaAsGF0DgOPaPwicr6WIeo0icDs2NS8qmN3Brwic9rAjoTibg/640?wx_fmt=png)

百度了下 getWritableDatabase 这个方法，这里需要一个 DBConstants.DB_PWD，那么目标就明确了，解出传入的值就可以得到解密密码。

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6wibpRQnlFPLyO69a9plJmYmauEOF4gHN9CicyYHSicx7v7Lqa2CaZpLVdV1V21MQX8MEIHzxZDSwhxWw/640?wx_fmt=png)

```
import java.security.MessageDigest;
import java.util.Base64;

public class 攻防世界_mobile_easyapk {
    public static void main(String[] args) {
        a();

    }

    public static void a() {
        // a2 是用户名Strange 和密码123456 分别前4位 然后做拼接。
        String a2 = "Stra1234"; // 这里我解出后直接把结果给a2 了。

        System.out.println(a2);

        String b2 = (a(a2 + b(a2, "123456")).substring(0, 7));
        System.out.println(b2);
    }

    public static String a(String str) {
        str = str + "yaphetshan";
        int i = 0;
        char[] cArr = new char[]{'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};
        try {
            byte[] bytes = str.getBytes();
            MessageDigest instance = MessageDigest.getInstance("SHA-1");
            return getString(i, cArr, bytes, instance);
        } catch (Exception e) {
            return null;
        }
    }

    private static String getString(int i, char[] cArr, byte[] bytes, MessageDigest instance) {
        instance.update(bytes);
        byte[] digest = instance.digest();
        int length = digest.length;
        char[] cArr2 = new char[(length * 2)];
        int i2 = 0;
        while (i < length) {
            byte b = digest[i];
            int i3 = i2 + 1;
            cArr2[i2] = cArr[(b >>> 4) & 15];
            i2 = i3 + 1;
            cArr2[i3] = cArr[b & 15];
            i++;
        }
        return new String(cArr2);
    }


    public static String b(String str, String str2) {
        int i = 0;
        char[] cArr = new char[]{'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};
        try {
            byte[] bytes = str.getBytes();
            MessageDigest instance = MessageDigest.getInstance("MD5");
            return getString(i, cArr, bytes, instance);
        } catch (Exception e) {
            return null;
        }
    }
}
```

搜索一下打开 db 文件的工具，我使用的是 SQLiteStudio ,

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6wibpRQnlFPLyO69a9plJmYmaVxDKvkuZnibZcoiaNiaZnEY0lL13SSOeX6UPyLMsWO5ia0iaYu9KiayJGxhg/640?wx_fmt=png)

然后找到 F_l_a_g 字段，取出来做一次 base64 解密即可获得 flag

![](https://mmbiz.qpic.cn/mmbiz_png/4ibic6bn2c2Uum6ptILukUMxBumOeMmibiaUzicL684EAic3lEIm9jBjynJ5G0uuJ8SJJZF5j5RJ2xvugicgxpeeNC45A/640?wx_fmt=png)

点个在看你最好看