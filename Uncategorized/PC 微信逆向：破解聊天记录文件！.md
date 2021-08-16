> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3tKTjbyNEkjwzhAfEClQjw)

在电子取证过程中，也会遇到提取 PC 版微信数据的情况，看雪、52 破解和 CSDN 等网上的 PC 版微信数据库破解文章实在是太简略了，大多数只有结果没有过程。经过反复试验终于成功解密了数据库，现在把详细过程记录下来，希望大家不要继续在已经解决的问题上过度浪费时间，以便更投入地研究尚未解决的问题。

通过查阅资料得知，与安卓手机版微信的 7 位密码不同，PC 版微信的密码是 32 字节（64 位），加密算法没有说明，但是可以通过 OllyDbg 工具从内存中获取到这个密码，然后通过一段 C++ 代码进行解密。

       首先下载 OllyDbg 2.01 汉化版，我用的版本如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8YKlF1ziaJcQOjWf6AN8SFgicEkhp6SFM1YnWZWyrBAVj8S4q24h0qD5Q/640?wx_fmt=png)  

       运行 OllyDbg，然后运行 PC 版微信（需要下载客户端的，不是网页版）。先不要点击登录按钮。

![](https://mmbiz.qpic.cn/mmbiz_jpg/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8LXehs79fx2AicOR76wKQZRhzm5pRpzwVAbjVC3LFEalKJ2ibSt9vBkGw/640?wx_fmt=jpeg)  

       切换到 Ollydbg 界面：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8dzpYTrr0cOOdpAWia9oRVVHvQFH3vDrg9uqTT1opVuTE8gnNcwZPhHQ/640?wx_fmt=png)  

       点击文件菜单，选择 “附加”，在弹出的对话框中找到名称为 WeChat 的进程，其窗口名称为 “登录”。然后点击 “附加”。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo86CLTxak0bQNnfwWQa0C8Yx4Zic4VgN7o6xYUNIFuoPmiciaGgfYRo2JibA/640?wx_fmt=png)  

       附加成功后 OllyDbg 开始加载，成功加载后可以看到最上面 OllyDbg 后面有 WeChat.exe 的字样：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8BMtUUwEpPSwuTFevU5L6xCftwMXK9UTbKaYvZxZVr7X5jrNth3xE4Q/640?wx_fmt=png)  

       在查看菜单中选择 “可执行模块”：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8ibK6DuxUj16mmibUqAI2CcS8MEW37PtT4FCHdoXUCEHVxmibEswULF3Vw/640?wx_fmt=png)  

       找到名称为 WeChatWin 的模块，双击选中。为了方便观察，在窗口菜单中选择水平平铺。在 CPU 窗口标题栏可以看到 “模块 WeChatWin” 字样。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8CE81A37s40VQby2rTBNeRro3cCH5J0q2PLpnmW87IOpc4SgR5q0KYw/640?wx_fmt=png)  

       在插件中选择 “StrFinder 字符查找” 中的“查找 ASCII 字符串”（注意如果下载的 OllyDbg 版本不对，可能没有相关插件，因此一定要找对版本），要稍微等一会儿，会出现搜索结果的窗口。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8iboSoJl1iccUFJqibXBMdk27AdDysBFRl0zRsyzg91O6xibZhCeKibwOeYA/640?wx_fmt=png)  

       在此窗口点击鼠标右键，选择 “Find”，在搜索框中输入 “DBFactory::encryptDB”。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8NU4Px1muj4H5W9nYkDc9pmSnD9MF36U6L6xBZXBNRxrpZeicXxLmwMg/640?wx_fmt=png)  

       会自动定位在第一处，但我们需要的是第二处，即 “encryptDB %s DBKey can’t be null” 下面这一处。可以用鼠标点击滚动条向下，找到第二处，用鼠标双击此处。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8vAVZSY2qiaIibL6oSESvEA3orpNrcQBwGgveiacMzJgRdyuTYFaSNgFcA/640?wx_fmt=png)  

       在 CPU 窗口中可以看到已经定位到了相应的位置。用鼠标点击滚动条向下翻。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8a44A4jKJuEmoNocv0BjhsTECeJQZe39NrW7N6AQbpP6OQvmhPHBegg/640?wx_fmt=png)  

       下面第六行应该是 TEST EDX,EDX，就是用来比对密码的汇编语言代码。在最前面地址位置（本文中是 0F9712BA）双击设置断点（设置断点成功则地址会被标红，而且可以在断点窗口中看到设置成功的断点）

       点击 “运行” 按钮（或者在调试菜单中选择“运行”），这时寄存器窗口中的 EDX 的值应该是 00000000。

       切换到微信登录页面，点击登录，然后到手机端确认登录。这是 OllyDbg 界面中的数据不断滚动，直到 EDX 不再为全 0 并且各个窗口内容停止滚动为止。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8890HBNFwhCX7jFdlPGJy5oSn1Ihtbyxg5NFjSjRa6ib0dgPm2Ekaz6w/640?wx_fmt=png)  

       在 EDX 的值上面点击鼠标右键，在弹出的菜单里面选择 “数据窗口中跟随”，则数据窗口中显示的就是 EDX 的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo80AcgoQVYy0fGwrDFgceNhVHp2uEmvHfCxsAqib0nM4MOuthHVwP3wQQ/640?wx_fmt=png)  

       图示中从 0B946A80（这个数值是变化的，不但每台电脑不同，每次调试也可能完全不同）到 0B946A9F 共 32 个字节就是微信的加密密码，本图中就是：

“53E9BFB23B724195A2BC6EB5BFEB0610DC2164756B9B4279BA32157639A40BB1”

       一共 32 个字节，共 64 位。

       得到这个之后，就可以关闭 OllyDbg 了，微信也会自动被关闭。

       接下来就是解密过程。在看雪、52 破解等多个论坛中都有相关的 C++ 源码，开始企图使用 Dev-C++ 或者 C-Free 等轻量级 IDE 进行编译，也使用过 Visual C++ 6.0 绿色精简版，结果多次尝试出现各种错误，反复失败，最终不得已使用 Visual Studio，并对代码进行了一定的修正，终于调试成功。

       正好 Visual Studio 2019 刚刚发布直接到官方网站下载了社区版。

       根据查到的资料，需要先安装 openssl，为了省事直接下载了最新的 Win64OpenSSL-1_1_1b，安装后发现各种报错，继续查找资料发现原来 sqlcipher 使用的是低版本的 openssl，之后找到了一个 Win64OpenSSL-1_0_2r 也报错，最后发现还是官方这个直接解压缩的版本靠谱：

       https://www.openssl.org/source/openssl-1.0.2r.tar.gz

       把压缩包直接解压到任意目录，比如 c:\openssl-1.0.2r

       启动 Visual Studio 2019 社区版（估计 Visual Studio 2008 以后的都应该可以，懒得找就直接官网下载最新的吧）

       在启动界面右下方选择 “创建新项目”

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8kXHanTVGibOiaeaN1oJibRWBkz8tv0zUrfrhiaMNcm9wMSicY0jsiawujNVA/640?wx_fmt=png)  

       滚动下拉条，在窗口中选择 C++ 控制台应用：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8QoNvq2ttYibEGXDM1GsYaCTmrx4VXHeyxveYqvRun3nQjYkfvywr1Gw/640?wx_fmt=png)  

       给项目随便起个名字，选择保存位置：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo81DNH8tMhShFicLuBicETXTdLicjaNKGkb42mxsPhQH2oJtxPdobAhxfiaA/640?wx_fmt=png)  

       然后点击 “创建”，即可完成新项目创建。生成默认的 Hello World 代码：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo85buAIEDFr0L7Lxgj7L2yrvPCTXW7qbRckL5QfrZ1y5dicKWJESMl3tg/640?wx_fmt=png)  

       先要做好项目的基础配置，之前调试失败主要问题就出在这里了。

       在项目菜单中最下面选择项目属性 “dewechat 属性”（这个跟设置的项目名称一致）

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8QuTvgJqMz1JooJHgEMicSjHmxnJRHKibHNJsPnoicmlOcTPLOicicodHulQ/640?wx_fmt=png)  

       对话框最左上角的配置后面，可以选择配置的是 Debug 模式还是 Release 模式（Release 模式不包含调试信息，编译完成的 exe 文件更小一些，但如果是自己用，这两个模式没有区别，配置了哪个，后面就要用哪个模式编译，否则会报错）

       先选择 C/C++ 下面的 “常规” 选项：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo86EeuoYaCbbPeSzRoORnqgPB82XxqdUHq0EO2jXZKpQgoFQJKuqanAg/640?wx_fmt=png)  

       右边第一条是 “附加包含目录”，点击右侧空白处。在下拉框里选择“编辑…”，在对话框中点击四个图标按钮最左侧的“新行” 按钮，会生成一个空白行，点击右侧的“…”：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8mHY1CByz1g37q6R4iarAg5blMIIleDVNaF7uY2l5Evq5uy9uIUhAiczA/640?wx_fmt=png)  

       在弹出的对话框里选择刚刚安装的 openssl 目录（本文是 c:\openssl-1.0.2r）中的 include 目录。  

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8LgEUiaUMxksG9VCyWTWYwX4YefibPSZfia9iaZFUiba6A4qFuC3iaPRY2qZQ/640?wx_fmt=png)  

       设置完成后如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8DzSmybNr4FSCDKtIia5TSicAaTlHMH4OFVmDKDqsKx0K7OxPbQLPVwDw/640?wx_fmt=png)  

       然后选择左侧 “链接器” 下面的“常规”：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8xT4oaCTIBrjUfCzLwnN0vUuypBCoaiaicqqlVXLICZH96cPibXKGMia4Vg/640?wx_fmt=png)  

       在中间位置，有一个 “附加库目录”，点击右侧空白处，选择 openssl 目录下的 lib 目录，设置完成后如下：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8QuLibqZgRV8oHsgGPEtK9TbWXqexhwjxRNdB1BdvCOFHbCu6ib5ypOVA/640?wx_fmt=png)  

       最后点击链接器下面的 “输入”：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8qh0jVkS7GefCDiaQfdoq70ULBZbhiclMYxYDnuoicEAGKxMGwZIxnqgkg/640?wx_fmt=png)  

       右侧最上面有 “附加依赖项”，默认已经有一些系统库，点击右侧内容，选择 “编辑…”

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo82PJ3H5jViaZNtZLVZqNksFVzVqT70lItdMn4YXpDibMuT4icjkSUxxZbQ/640?wx_fmt=png)  

       这个没有增加新行的按钮，只能手工录入或者拷贝文件名进去，需要增加上图所示的两个库名称。

       设置完成后如下：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8n4Q5KZZUBH0O1Wa4ENbY91alzLY1HQgaq8mf1Fmsp8eBxQaUWAejuQ/640?wx_fmt=png)  

       现在所有的设置都 OK 了，可以把代码放进来编译了。

       由于太多网站转载，而且很多有错漏，已经搞不清原始代码是哪位大神写的了，其中有一些已经被废弃的代码，根据系统报错提示进行了替换，另外做了一个主要的变化就是之前的代码是把数据库名写在变量中，但由于需要解密很多库，为了灵活，改为输入参数的方法，即在运行时带参数运行或者根据提示输入需要解密的数据库文件名。

```
using namespace std;
#include <Windows.h>
#include <iostream>
#include <openssl/rand.h>
#include <openssl/evp.h>
#include <openssl/aes.h>
#include <openssl/hmac.h>
 
#undef _UNICODE
#define SQLITE_FILE_HEADER "SQLite format 3" 
#define IV_SIZE 16
#define HMAC_SHA1_SIZE 20
#define KEY_SIZE 32
 
#define SL3SIGNLEN 20
 
#ifndef ANDROID_WECHAT
#define DEFAULT_PAGESIZE 4096       //4048数据 + 16IV + 20 HMAC + 12
#define DEFAULT_ITER 64000
#else
#define NO_USE_HMAC_SHA1
#define DEFAULT_PAGESIZE 1024
#define DEFAULT_ITER 4000
#endif
//pc端密码是经过OllyDbg得到的32位pass。
unsigned char pass[] = { 0x53,0xE9,0xBF,0xB2,0x3B,0x72,0x41,0x95,0xA2,0xBC,0x6E,0xB5,0xBF,0xEB,0x06,0x10,0xDC,0x21,0x64,0x75,0x6B,0x9B,0x42,0x79,0xBA,0x32,0x15,0x76,0x39,0xA4,0x0B,0xB1 };
char dbfilename[50];
int Decryptdb();
int CheckKey();
int CheckAESKey();
int main(int argc, char* argv[])
{
    if (argc >= 2)    //第二个参数argv[1]是文件名
        strcpy_s(dbfilename, argv[1]);  //复制    
           //没有提供文件名，则提示用户输入
    else {
        cout << "请输入文件名:" << endl;
        cin >> dbfilename;
    }
    Decryptdb();
    return 0;
}
 
int Decryptdb()
{
    FILE* fpdb;
    fopen_s(&fpdb, dbfilename, "rb+");
    if (!fpdb)
    {
        printf("打开文件错!");
        getchar();
        return 0;
    }
    fseek(fpdb, 0, SEEK_END);
    long nFileSize = ftell(fpdb);
    fseek(fpdb, 0, SEEK_SET);
    unsigned char* pDbBuffer = new unsigned char[nFileSize];
    fread(pDbBuffer, 1, nFileSize, fpdb);
    fclose(fpdb);
 
    unsigned char salt[16] = { 0 };
    memcpy(salt, pDbBuffer, 16);
 
#ifndef NO_USE_HMAC_SHA1
    unsigned char mac_salt[16] = { 0 };
    memcpy(mac_salt, salt, 16);
    for (int i = 0; i < sizeof(salt); i++)
    {
        mac_salt[i] ^= 0x3a;
    }
#endif
 
    int reserve = IV_SIZE;      //校验码长度,PC端每4096字节有48字节
#ifndef NO_USE_HMAC_SHA1
    reserve += HMAC_SHA1_SIZE;
#endif
    reserve = ((reserve % AES_BLOCK_SIZE) == 0) ? reserve : ((reserve / AES_BLOCK_SIZE) + 1) * AES_BLOCK_SIZE;
 
    unsigned char key[KEY_SIZE] = { 0 };
    unsigned char mac_key[KEY_SIZE] = { 0 };
 
    OpenSSL_add_all_algorithms();
    PKCS5_PBKDF2_HMAC_SHA1((const char*)pass, sizeof(pass), salt, sizeof(salt), DEFAULT_ITER, sizeof(key), key);
#ifndef NO_USE_HMAC_SHA1
    PKCS5_PBKDF2_HMAC_SHA1((const char*)key, sizeof(key), mac_salt, sizeof(mac_salt), 2, sizeof(mac_key), mac_key);
#endif
 
    unsigned char* pTemp = pDbBuffer;
    unsigned char pDecryptPerPageBuffer[DEFAULT_PAGESIZE];
    int nPage = 1;
    int offset = 16;
    while (pTemp < pDbBuffer + nFileSize)
    {
        printf("解密数据页:%d/%d \n", nPage, nFileSize / DEFAULT_PAGESIZE);
 
#ifndef NO_USE_HMAC_SHA1
        unsigned char hash_mac[HMAC_SHA1_SIZE] = { 0 };
        unsigned int hash_len = 0;
        HMAC_CTX hctx;
        HMAC_CTX_init(&hctx);
        HMAC_Init_ex(&hctx, mac_key, sizeof(mac_key), EVP_sha1(), NULL);
        HMAC_Update(&hctx, pTemp + offset, DEFAULT_PAGESIZE - reserve - offset + IV_SIZE);
        HMAC_Update(&hctx, (const unsigned char*)& nPage, sizeof(nPage));
        HMAC_Final(&hctx, hash_mac, &hash_len);
        HMAC_CTX_cleanup(&hctx);
        if (0 != memcmp(hash_mac, pTemp + DEFAULT_PAGESIZE - reserve + IV_SIZE, sizeof(hash_mac)))
        {
            printf("\n 哈希值错误! \n");
            getchar();
            return 0;
        }
#endif
        //
        if (nPage == 1)
        {
            memcpy(pDecryptPerPageBuffer, SQLITE_FILE_HEADER, offset);
        }
 
        EVP_CIPHER_CTX* ectx = EVP_CIPHER_CTX_new();
        EVP_CipherInit_ex(ectx, EVP_get_cipherbyname("aes-256-cbc"), NULL, NULL, NULL, 0);
        EVP_CIPHER_CTX_set_padding(ectx, 0);
        EVP_CipherInit_ex(ectx, NULL, NULL, key, pTemp + (DEFAULT_PAGESIZE - reserve), 0);
 
        int nDecryptLen = 0;
        int nTotal = 0;
        EVP_CipherUpdate(ectx, pDecryptPerPageBuffer + offset, &nDecryptLen, pTemp + offset, DEFAULT_PAGESIZE - reserve - offset);
        nTotal = nDecryptLen;
        EVP_CipherFinal_ex(ectx, pDecryptPerPageBuffer + offset + nDecryptLen, &nDecryptLen);
        nTotal += nDecryptLen;
        EVP_CIPHER_CTX_free(ectx);
 
        memcpy(pDecryptPerPageBuffer + DEFAULT_PAGESIZE - reserve, pTemp + DEFAULT_PAGESIZE - reserve, reserve);
        char decFile[1024] = { 0 };
        sprintf_s(decFile, "dec_%s", dbfilename);
        FILE * fp;
        fopen_s(&fp, decFile, "ab+");
        {
            fwrite(pDecryptPerPageBuffer, 1, DEFAULT_PAGESIZE, fp);
            fclose(fp);
        }
 
        nPage++;
        offset = 0;
        pTemp += DEFAULT_PAGESIZE;
    }
    printf("\n 解密成功! \n");
    return 0;
}
```

 将之前默认的代码全部清除，将以上代码拷贝进去，保存。然后在工具条栏中选择是 Debug 还是 Release 模式，是 x86 还是 x64（需要跟之前配置匹配，如果选了没配置的模式会报错。测试发现几个选项没有太大区别，建议默认），之后点击 “本地 windows 调试器”（或者按 F5 键），如果前面的步骤操作都正确，应该可以完成编译并自动运行，弹出一个命令行窗口，提示需要输入文件名：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8JTl1k8NQkLYNZAPYFBQibgoNIJ7FdE09c7Bsy9HeyGYT0Nicgt9PbXZA/640?wx_fmt=png)  

       最下方显示了生成的 exe 文件路径，将这个文件拷贝到微信数据库所在的目录，一般是：

C:\Users\Administrator\Documents\WeChat Files\********\Msg

       其中 ******** 位置为需要解密的微信 id，目录内容如下：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8xicNNarqYgUb4pOtHgYWblnStGicq1YLTqjAIrg3LAWdQyyjKtnia0WjQ/640?wx_fmt=png)  

       如果要解密 ChatMsg.db，则在命令行窗口输入指令 dewechat ChatMsg.db 回车即可。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdlS6dvYMN7fjWBoFhUFVo8SxwYGlaO7Vv8UEXibOSGMw8ztPLvANfrVZrI27gFJibPLGwzw5ibDeDfQ/640?wx_fmt=png)  

       解密成功后，会在目录中生成 de_ChatMsg.db，用 sqlite 数据库管理软件打开即可。

       本文主要是个验证过程，没有做什么突破工作，目前的解密只能算是半自动过程，密码算法部分的获得是下一步需要研究的内容，希望大家共同努力！

> 原文链接：https://bbs.pediy.com/thread-251303.htm
> 
> 作者：newx