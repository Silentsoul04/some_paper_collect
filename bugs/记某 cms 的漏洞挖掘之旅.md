> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ETm92MHTNksURjOPNqFgHg)

‍‍

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

原创稿件征集

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬

‍

任意文件写入
------

这个 cms 是基于 thinkphp5.1 的基础开发的，一般我们挖 cms 如果想 rce 的话，可以在 application 文件夹直接搜索`file_put_content`等危险函数，如下图，我们直接全局定位到这个`fileedit`方法里面的`file_put_content`

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKppicOWV5N7BjzuptFMDh5PxOX0WmbXrwiamqXbDupu7EmFZxCVibxiaM2Q/640?wx_fmt=png)

我们看到第一个参数`$rootpath`，他是被拼接了这么一段路径  

```
$rootpath = Env::get('root_path') . 'theme' . DIRECTORY_SEPARATOR . $template . DIRECTORY_SEPARATOR . $path;


```

其中`$path`是我们可控的，那么一般就可以考虑下是否存在路径穿越的问题

再看到第二个参数`htmlspecialchars_decode(Request::param('html')`也是我们可控的

所以这里就比较清晰了，我们只需要`../`就可以进行路径穿越，`htmlspecialchars_decode`也对我们写入 php 代码没有什么影响，所以我们直接 post 传参 `path=../../index.php&html=<?php phpinfo();?>`即可

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKlAAVHZtfcQW6wia87L2IM0IDvtIiaj3iblCMt9OV1tjb6SJdsVl9FtmYg/640?wx_fmt=png)

可以看到已经成功 rce

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysK0SSYUgtCYT7yoxSwTQRE8HOJjEM29YmCmrs9DRxsSMnmXZLWovY1WQ/640?wx_fmt=png)

任意文件读取
------

我们再顺着`fileedit`这个方法往下瞅瞅，发现还有一个`file_get_contents`，他的参数也是`$rootpath`，所以这里也是我们可控的，不同的是进入这个 else 分支我们用 get 传参即可

我们直接传入`../../index.php`，发现已经成功把`index.php`读取出来了

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKY0ZP9XUfSlFvepMCPGib0aFNeuqddV7mbibUyXX1wuElfgXdW2I5R00Q/640?wx_fmt=png)

反序列化漏洞
------

上面两个漏洞是利用了`file_get_contents`和`file_put_content`，这两个函数都是涉及了 IO 的操作函数，也就是说可以进行操作 phar 反序列化漏洞，但是他们的路径并不是完全可控的，只是后面一小部分可控，所以这条路走不通，所以接下来的思路就是搜索有没有可以操作`phar`的函数

我们直接全局搜索`is_dir`，一个一个分析是否可以利用

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKYFBSLqDKv7IOuVXAyMc88hRibN5IrgRWCCxBdsfHW1Hcwk6H1eoAr8A/640?wx_fmt=png)

这里我的运气比较好，映入眼帘的是`scanFilesForTree`这个方法，他的`$dir`是直接可控的，文章的开头说了这个 cms 是基于 thinkphp5.1 二次开发的，所以我们可以直接利用这个漏洞生成 phar 文件来进行 rce

我们首先看看能不能上传 phar 文件，在后台一处发现可以上传文件

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKibkJzJUWoaArXUeClaecdorGxs0yMaOOvR5pThiatRKnKdibw2x5J9HJg/640?wx_fmt=png)

我们先抓个包试试水，发现提示非法图片文件，应该是写了什么过滤

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKdLXGAqSBgJRAxjt3b3T05XGUxYzQVDiaOa7PVcxfIkpY01kdc0rKxrw/640?wx_fmt=png)

我们找到`upload`这个函数发现对图片的类型和大小进行了一些验证

```
public function upload($file, $fileType = 'image')
    {
        // 验证文件类型及大小
        switch ($fileType)
        {
            case 'image':
                $result = $file->check(['ext' => $this->config['upload_image_ext'], 'size' => $this->config['upload_image_size']*1024]);
                if(empty($result)){
                    // 上传失败获取错误信息
                    $this->error = $file->getError();
                    return false;
                }
                break;
        $result =  $this->uploadHandler->upload($file);
        $data   =  array_merge($result, ['site_id' => $this->site_id]);
        SiteFile::create($data);
        return $data;
    }


```

然后尝试加了 GIF89a 头就可以上传了，看来多打 CTF 还是有用的，于是直接上传我们的 phar 文件就好了

这里要记得生成 phar 文件的时候要要加入 GIF89a 头来绕过，如下

```
$phar->setStub('GIF89a'.'<?php __HALT_COMPILER();?>');//设置stub


```

可以看到已经成功上传了，同时记住下面那个路径

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysK1h1OZOZZtQlR8gwkgFnicicyatW28ticNmA0A15pGibzejV5TRoJw0w6zQ/640?wx_fmt=png)

最后我们在`scanFilesForTree`这里触发我们的`phar`文件就可以了

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LcQNR6ddraoWCG6k9kRwysKgqscB3emzWQtHLJZ0jDKgcWuA4SPWp2atraIN076QI4fjUOp2LAMZQ/640?wx_fmt=png)

总结
--

本篇的漏洞已经全部上交 cnvd，这个 cms 总的来说比较适合练手，主要的切入点还是通过白盒通过寻找一些危险的函数，再想方设法的去控制它的参数变量

**知识点实操，复制链接开始操作 -** 任意文件下载漏洞的代码审计  

**https://www.hetianlab.com/expc.do?ec=ECID06a1-2876-4bfb-8e59-a0096299c167********&pk_campaign=weixin-wemedia#stu********** 

本节的学习，了解文件下载漏洞的原理，通过代码审计掌握文件下载漏洞产生的原因以及修复方法。 

  

向知识的海洋进军，领取 39 元会员卡。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LeSW1juVkgMv4lIFiaGCGQ3Ria2H430QJC2VVT8faPBOxtBjY2w5htSbpotFibibbwXbG2f9GcZ7lUhAA/640?wx_fmt=png)