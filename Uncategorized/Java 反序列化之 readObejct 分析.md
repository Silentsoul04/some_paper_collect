> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0jaMflNV01gAG_w2VLGaug)

前言
--

之前介绍 java 反序列化的时候已经对 readObject 有了一个大概的了解，知道 Java 在将字节数据反序列化为 Java 对象的时候会调用流对象的 readObject。

本文较长，可以直接拉到尾部，查看 readObject 的执行流程图

readObject 分析
-------------

例子 ：`Java 反序列化-开篇`中第一个代码

```
import java.io.*;

public class test{
    public static void main(String args[]) throws Exception{
        //定义myObj对象
        MyObject myObj = new MyObject();
        myObj.name = "hi";
        //创建一个包含对象进行反序列化信息的”object”数据文件
        FileOutputStream fos = new FileOutputStream("object");
        ObjectOutputStream os = new ObjectOutputStream(fos);
        //writeObject()方法将myObj对象写入object文件
        os.writeObject(myObj);
        os.close();
        //从文件中反序列化obj对象
        FileInputStream fis = new FileInputStream("object");
        ObjectInputStream ois = new ObjectInputStream(fis);
        //恢复对象
        MyObject objectFromDisk = (MyObject)ois.readObject();
        System.out.println(objectFromDisk.name);
        ois.close();
    }
}

class MyObject implements Serializable {
    public String name;
    //重写readObject()方法
    private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException{
        //执行默认的readObject()方法
        in.defaultReadObject();
        //执行打开计算器程序命令
        Runtime.getRuntime().exec("open -a Calculator");
    }
}
```

从`ObjectInputStream.readObject()`跟进代码

```
public final Object readObject()
    throws IOException, ClassNotFoundException {
    return readObject(Object.class);
}
```

这里直接 返回了 readObject(Object.class) 继续跟进

```
private final Object readObject(Class<?> type)
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        if (! (type == Object.class || type == String.class))
            throw new AssertionError("internal error");

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            Object obj = readObject0(type, false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
            }
            return obj;
        } finally {
            passHandle = outerHandle;
            if (closed && depth == 0) {
                clear();
            }
        }
    }
```

开头先判断 `enableOverride` ，如果为 true 调用`readObjectOverride()` 而不是`readObject()`  

```
public ObjectInputStream(InputStream in) throws IOException {
    ...
    enableOverride = false;
    ...
}
```

只有在构造函数实例化时不带参数，才能使`enableOverride` 为 true

```
protected ObjectInputStream() throws IOException, SecurityException {
    ...
    enableOverride = true;
}
```

当 `enableOverride` 为 true 时，跟进 readObjectOverride 函数

```
protected Object readObjectOverride()
    throws IOException, ClassNotFoundException
{
    return null;
}
```

我们的实例化方法`ObjectInputStream` 是有参数的，所以不会进入`readObjectOverride`, 继续往下分析

```
Object obj = readObject0(type, false);
```

从`readObject0`跟进去

```
private Object readObject0(Class<?> type, boolean unshared) throws IOException {
    boolean oldMode = bin.getBlockDataMode();
    if (oldMode) {
        int remain = bin.currentBlockRemaining();
        if (remain > 0) {
            throw new OptionalDataException(remain);
        } else if (defaultDataEnd) {
            /*
             * Fix for 4360508: stream is currently at the end of a field
             * value block written via default serialization; since there
             * is no terminating TC_ENDBLOCKDATA tag, simulate
             * end-of-custom-data behavior explicitly.
             */
            throw new OptionalDataException(true);
        }
        bin.setBlockDataMode(false);
    }

    byte tc;
    while ((tc = bin.peekByte()) == TC_RESET) {
        bin.readByte();
        handleReset();
    }

    depth++;
    totalObjectRefs++;
    try {
        switch (tc) {
            case TC_NULL:
                return readNull();

            case TC_REFERENCE:
                // check the type of the existing object
                return type.cast(readHandle(unshared));

            case TC_CLASS:
                if (type == String.class) {
                    throw new ClassCastException("Cannot cast a class to java.lang.String");
                }
                return readClass(unshared);

            case TC_CLASSDESC:
            case TC_PROXYCLASSDESC:
                if (type == String.class) {
                    throw new ClassCastException("Cannot cast a class to java.lang.String");
                }
                return readClassDesc(unshared);

            case TC_STRING:
            case TC_LONGSTRING:
                return checkResolve(readString(unshared));

            case TC_ARRAY:
                if (type == String.class) {
                    throw new ClassCastException("Cannot cast an array to java.lang.String");
                }
                return checkResolve(readArray(unshared));

            case TC_ENUM:
                if (type == String.class) {
                    throw new ClassCastException("Cannot cast an enum to java.lang.String");
                }
                return checkResolve(readEnum(unshared));

            case TC_OBJECT:
                if (type == String.class) {
                    throw new ClassCastException("Cannot cast an object to java.lang.String");
                }
                return checkResolve(readOrdinaryObject(unshared));

            case TC_EXCEPTION:
                if (type == String.class) {
                    throw new ClassCastException("Cannot cast an exception to java.lang.String");
                }
                IOException ex = readFatalException();
                throw new WriteAbortedException("writing aborted", ex);

            case TC_BLOCKDATA:
            case TC_BLOCKDATALONG:
                if (oldMode) {
                    bin.setBlockDataMode(true);
                    bin.peek();             // force header read
                    throw new OptionalDataException(
                        bin.currentBlockRemaining());
                } else {
                    throw new StreamCorruptedException(
                        "unexpected block data");
                }

            case TC_ENDBLOCKDATA:
                if (oldMode) {
                    throw new OptionalDataException(true);
                } else {
                    throw new StreamCorruptedException(
                        "unexpected end of block data");
                }

            default:
                throw new StreamCorruptedException(
                    String.format("invalid type code: %02X", tc));
        }
    } finally {
        depth--;
        bin.setBlockDataMode(oldMode);
    }
}
```

这个`readObject0`是底层`readObject` 的实现，开头的`bin`没有在函数中生成，跟一下`bin`

```
public ObjectInputStream(InputStream in) throws IOException {
    ...
    bin = new BlockDataInputStream(in);
    ...
    bin.setBlockDataMode(true);
}
```

`bin`是在`ObjectInputStream`初始化的时候生成的，现在还不知道`bin`的用处，继续跟`BlockDataInputStream`  

`BlockDataInputStream`是`ObjectInputStream` 块数据输入流，用来完成对序列化 Stream 的读取，其分为两种模式：`Default mode` 和`Block mode` ，默认关闭`Block mode`

回到上层代码，如果采用块数据模式，`bin.getBlockDataMode()`就为 true 否则为 false

如果是块数据模式，就检测当前数据块中剩余未消耗的字节数，如果不是就跑出错误，然后就就转到`defalutDataEnd`，重新设置块数据模式为 false

```
byte tc;
while ((tc = bin.peekByte()) == TC_RESET) {
    bin.readByte();
    handleReset();
}
```

继续分析`readObject0`中这一部分代码，`tc` 会获取流的下一个字节值

继续分析遇到`switch`判断，通过调试或者分析生成的序列化文件`object`可以知道`tc`会走到`TC_OBJECT`这个分支

```
case TC_OBJECT:
    if (type == String.class) {
        throw new ClassCastException("Cannot cast an object to java.lang.String");
    }
    return checkResolve(readOrdinaryObject(unshared));
```

跟进`readOrdinaryObject(unshared)`

```
private Object readOrdinaryObject(boolean unshared)
    throws IOException
{
    if (bin.readByte() != TC_OBJECT) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    desc.checkDeserialize();

    Class<?> cl = desc.forClass();
    if (cl == String.class || cl == Class.class
            || cl == ObjectStreamClass.class) {
        throw new InvalidClassException("invalid class descriptor");
    }

    Object obj;
    try {
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(
            desc.forClass().getName(),
            "unable to create instance").initCause(ex);
    }

    passHandle = handles.assign(unshared ? unsharedMarker : obj);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(passHandle, resolveEx);
    }

    if (desc.isExternalizable()) {
        readExternalData((Externalizable) obj, desc);
    } else {
        readSerialData(obj, desc);
    }

    handles.finish(passHandle);

    if (obj != null &&
        handles.lookupException(passHandle) == null &&
        desc.hasReadResolveMethod())
    {
        Object rep = desc.invokeReadResolve(obj);
        if (unshared && rep.getClass().isArray()) {
            rep = cloneArray(rep);
        }
        if (rep != obj) {
            // Filter the replacement object
            if (rep != null) {
                if (rep.getClass().isArray()) {
                    filterCheck(rep.getClass(), Array.getLength(rep));
                } else {
                    filterCheck(rep.getClass(), -1);
                }
            }
            handles.setObject(passHandle, obj = rep);
        }
    }

    return obj;
}
```

开头先做了简单的判断，之后调用方法`readClassDesc`

```
private ObjectStreamClass readClassDesc(boolean unshared)
        throws IOException
{
        byte tc = bin.peekByte();
        ObjectStreamClass descriptor;
        switch (tc) {
            case TC_NULL:
                descriptor = (ObjectStreamClass) readNull();
                break;
            case TC_REFERENCE:
                descriptor = (ObjectStreamClass) readHandle(unshared);
                // Should only reference initialized class descriptors
                descriptor.checkInitialized();
                break;
            case TC_PROXYCLASSDESC:
                descriptor = readProxyDesc(unshared);
                break;
            case TC_CLASSDESC:
                descriptor = readNonProxyDesc(unshared);
                break;
            default:
                throw new StreamCorruptedException(
                    String.format("invalid type code: %02X", tc));
        }
        if (descriptor != null) {
            validateDescriptor(descriptor);
        }
        return descriptor;
    }
```

读取并返回类描述符，在文件格式分析那篇文章中，分析了`object`的文件格式，读到的第一个 Byte 是 `0x72`，用来描述类的结构，类名，成员类型等

跟进去`descriptor = readNonProxyDesc(unshared)`

```
private ObjectStreamClass readNonProxyDesc(boolean unshared)
    throws IOException
{
    if (bin.readByte() != TC_CLASSDESC) {
        throw new InternalError();
    }

    ObjectStreamClass desc = new ObjectStreamClass();
    int descHandle = handles.assign(unshared ? unsharedMarker : desc);
    passHandle = NULL_HANDLE;

    ObjectStreamClass readDesc = null;
    try {
        readDesc = readClassDescriptor();
    } catch (ClassNotFoundException ex) {
        throw (IOException) new InvalidClassException(
            "failed to read class descriptor").initCause(ex);
    }

    Class<?> cl = null;
    ClassNotFoundException resolveEx = null;
    bin.setBlockDataMode(true);
    final boolean checksRequired = isCustomSubclass();
    try {
        if ((cl = resolveClass(readDesc)) == null) {
            resolveEx = new ClassNotFoundException("null class");
        } else if (checksRequired) {
            ReflectUtil.checkPackageAccess(cl);
        }
    } catch (ClassNotFoundException ex) {
        resolveEx = ex;
    }

    // Call filterCheck on the class before reading anything else
    filterCheck(cl, -1);

    skipCustomData();

    try {
        totalObjectRefs++;
        depth++;
        desc.initNonProxy(readDesc, cl, resolveEx, readClassDesc(false));
    } finally {
        depth--;
    }

    handles.finish(descHandle);
    passHandle = descHandle;

    return desc;
}
```

读入并返回非动态代理类的类描述符, 首先初始化`ObjectStreamClass`给`desc`，然后通过`readClassDescriptor()`去初始化`ObjectStreamClass`，在通过`readNonProxy(this)`来读取非代理类描述符信息

```
void readNonProxy(ObjectInputStream in)
    throws IOException, ClassNotFoundException
{
    name = in.readUTF();
    suid = Long.valueOf(in.readLong());
    isProxy = false;

    byte flags = in.readByte();
    hasWriteObjectData =
        ((flags & ObjectStreamConstants.SC_WRITE_METHOD) != 0);
    hasBlockExternalData =
        ((flags & ObjectStreamConstants.SC_BLOCK_DATA) != 0);
    externalizable =
        ((flags & ObjectStreamConstants.SC_EXTERNALIZABLE) != 0);
    boolean sflag =
        ((flags & ObjectStreamConstants.SC_SERIALIZABLE) != 0);
    if (externalizable && sflag) {
        throw new InvalidClassException(
            name, "serializable and externalizable flags conflict");
    }
    serializable = externalizable || sflag;
    isEnum = ((flags & ObjectStreamConstants.SC_ENUM) != 0);
    if (isEnum && suid.longValue() != 0L) {
        throw new InvalidClassException(name,
            "enum descriptor has non-zero serialVersionUID: " + suid);
    }

    int numFields = in.readShort();
    if (isEnum && numFields != 0) {
        throw new InvalidClassException(name,
            "enum descriptor has non-zero field count: " + numFields);
    }
    fields = (numFields > 0) ?
        new ObjectStreamField[numFields] : NO_FIELDS;
    for (int i = 0; i < numFields; i++) {
        char tcode = (char) in.readByte();
        String fname = in.readUTF();
        String signature = ((tcode == 'L') || (tcode == '[')) ?
            in.readTypeString() : new String(new char[] { tcode });
        try {
            fields[i] = new ObjectStreamField(fname, signature, false);
        } catch (RuntimeException e) {
            throw (IOException) new InvalidClassException(name,
                "invalid descriptor for field " + fname).initCause(e);
        }
    }
    computeFieldOffsets();
}
```

从这里就是对输入流进行细致的判断读取了，  

name 就是 `class descriptor`的`class name`

suid 就是 `serialVersionUID` 用来验证该类是否被修改过的验证码, 至于`serialversionUID`是怎么生成的 可以在我`java 反序列化writeObject分析`中查看

`readNonProxy()`初始化类名，suid 之后，`readClassDescriptor`将得到的类描述符返回后，继续往下分析

```
private ObjectStreamClass readNonProxyDesc(boolean unshared)
    throws IOException
    ... 

    Class<?> cl = null;
    ClassNotFoundException resolveEx = null;
    bin.setBlockDataMode(true);
    final boolean checksRequired = isCustomSubclass();
    try {
        if ((cl = resolveClass(readDesc)) == null) {
            resolveEx = new ClassNotFoundException("null class");
        } else if (checksRequired) {
            ReflectUtil.checkPackageAccess(cl);
        }
    } catch (ClassNotFoundException ex) {
        resolveEx = ex;
    }

    // Call filterCheck on the class before reading anything else
    filterCheck(cl, -1);

    skipCustomData();

    try {
        totalObjectRefs++;
        depth++;
        desc.initNonProxy(readDesc, cl, resolveEx, readClassDesc(false));
    } finally {
        depth--;
    }

    handles.finish(descHandle);
    passHandle = descHandle;

    return desc;
}
```

类描述符信息 `readDesc`传入了`resolveClass`

```
protected Class<?> resolveClass(ObjectStreamClass desc)
    throws IOException, ClassNotFoundException
{
    String name = desc.getName();
    try {
        return Class.forName(name, false, latestUserDefinedLoader());
    } catch (ClassNotFoundException ex) {
        Class<?> cl = primClasses.get(name);
        if (cl != null) {
            return cl;
        } else {
            throw ex;
        }
    }
}
```

这个方法 加载与指定流描述符等价的局部类，也就是通过 java 反射获取当前描述的类别信息。

> Java 反射 具体参见 《Java 反射》
> 
> 在运行时动态的创建对象并调用其属性，不需要在编译期知道运行的对象是谁

调用`filterCheck`检查获得的描述符`cl`

```
private void filterCheck(Class<?> clazz, int arrayLength)
        throws InvalidClassException {
    if (serialFilter != null) {
        RuntimeException ex = null;
        ObjectInputFilter.Status status;
        // Info about the stream is not available if overridden by subclass, return 0
        long bytesRead = (bin == null) ? 0 : bin.getBytesRead();
        try {
            status = serialFilter.checkInput(new FilterValues(clazz, arrayLength,
                    totalObjectRefs, depth, bytesRead));
        } catch (RuntimeException e) {
            // Preventive interception of an exception to log
            status = ObjectInputFilter.Status.REJECTED;
            ex = e;
        }
        if (status == null  ||
                status == ObjectInputFilter.Status.REJECTED) {
            // Debug logging of filter checks that fail
            if (Logging.infoLogger != null) {
                Logging.infoLogger.info(
                        "ObjectInputFilter {0}: {1}, array length: {2}, nRefs: {3}, depth: {4}, bytes: {5}, ex: {6}",
                        status, clazz, arrayLength, totalObjectRefs, depth, bytesRead,
                        Objects.toString(ex, "n/a"));
            }
            InvalidClassException ice = new InvalidClassException("filter status: " + status);
            ice.initCause(ex);
            throw ice;
        } else {
            // Trace logging for those that succeed
            if (Logging.traceLogger != null) {
                Logging.traceLogger.finer(
                        "ObjectInputFilter {0}: {1}, array length: {2}, nRefs: {3}, depth: {4}, bytes: {5}, ex: {6}",
                        status, clazz, arrayLength, totalObjectRefs, depth, bytesRead,
                        Objects.toString(ex, "n/a"));
            }
        }
    }
}
```

`serialFilter`是在`objectInputStream`初始化时生成的

当`serialFilter`存在时，会对信息流进行检测，如果没有通过返回报错

检查结束后，继续回到`readNonProxyDesc()`

```
private ObjectStreamClass readNonProxyDesc(boolean unshared)
    throws IOException
{
    ...

    skipCustomData();

    try {
        totalObjectRefs++;
        depth++;
        desc.initNonProxy(readDesc, cl, resolveEx, readClassDesc(false));
    } finally {
        depth--;
    }

    handles.finish(descHandle);
    passHandle = descHandle;

    return desc;
}
```

跟进`skipCustmoData()`

```
private void skipCustomData() throws IOException {
    int oldHandle = passHandle;
    for (;;) {
        if (bin.getBlockDataMode()) {
            bin.skipBlockData();
            bin.setBlockDataMode(false);
        }
        switch (bin.peekByte()) {
            case TC_BLOCKDATA:
            case TC_BLOCKDATALONG:
                bin.setBlockDataMode(true);
                break;

            case TC_ENDBLOCKDATA:
                bin.readByte();
                passHandle = oldHandle;
                return;

            default:
                readObject0(Object.class, false);
                break;
        }
    }
}
```

跳过所有块数据和对象，直到遇到 TC_ENDBLOCKDATA

继续分析`readNonPorxyDesc`, 跟进`initNonPorxy`

```
void initNonProxy(ObjectStreamClass model,
                  Class<?> cl,
                  ClassNotFoundException resolveEx,
                  ObjectStreamClass superDesc)
    throws InvalidClassException
{
     long suid = Long.valueOf(model.getSerialVersionUID());
        ObjectStreamClass osc = null;
        if (cl != null) {
            osc = lookup(cl, true);
            if (osc.isProxy) {
                throw new InvalidClassException(
                        "cannot bind non-proxy descriptor to a proxy class");
            }
            if (model.isEnum != osc.isEnum) {
                throw new InvalidClassException(model.isEnum ?
                        "cannot bind enum descriptor to a non-enum class" :
                        "cannot bind non-enum descriptor to an enum class");
            }

            if (model.serializable == osc.serializable &&
                    !cl.isArray() &&
                    suid != osc.getSerialVersionUID()) {
                throw new InvalidClassException(osc.name,
                        "local class incompatible: " +
                                "stream classdesc serialVersionUID = " + suid +
                                ", local class serialVersionUID = " +
                                osc.getSerialVersionUID());
            }

            if (!classNamesEqual(model.name, osc.name)) {
                throw new InvalidClassException(osc.name,
                        "local class name incompatible with stream class " +
                                "name \"" + model.name + "\"");
            }

            if (!model.isEnum) {
                if ((model.serializable == osc.serializable) &&
                        (model.externalizable != osc.externalizable)) {
                    throw new InvalidClassException(osc.name,
                            "Serializable incompatible with Externalizable");
                }

                if ((model.serializable != osc.serializable) ||
                        (model.externalizable != osc.externalizable) ||
                        !(model.serializable || model.externalizable)) {
                    deserializeEx = new ExceptionInfo(
                            osc.name, "class invalid for deserialization");
                }
            }
        }

    this.cl = cl;
    this.resolveEx = resolveEx;
    this.superDesc = superDesc;
    name = model.name;
    this.suid = suid;
    isProxy = false;
    isEnum = model.isEnum;
    serializable = model.serializable;
    externalizable = model.externalizable;
    hasBlockExternalData = model.hasBlockExternalData;
    hasWriteObjectData = model.hasWriteObjectData;
    fields = model.fields;
    primDataSize = model.primDataSize;
    numObjFields = model.numObjFields;

    if (osc != null) {
        localDesc = osc;
        writeObjectMethod = localDesc.writeObjectMethod;
        readObjectMethod = localDesc.readObjectMethod;
        readObjectNoDataMethod = localDesc.readObjectNoDataMethod;
        writeReplaceMethod = localDesc.writeReplaceMethod;
        readResolveMethod = localDesc.readResolveMethod;
        if (deserializeEx == null) {
            deserializeEx = localDesc.deserializeEx;
        }
        domains = localDesc.domains;
        cons = localDesc.cons;
    }

    fieldRefl = getReflector(fields, localDesc);
    // reassign to matched fields so as to reflect local unshared settings
    fields = fieldRefl.getFields();
    initialized = true;
}
```

这个方法会使用`readDesc`来初始化`Desc`, 所以开头先检查`readDesc`的正确性

使用本地直接 new 的描述 与`model`也就是`readDesc`里的内容进行判断，如果不同就抛出错误

继续往下看`initNonProxy()`

把`localDesc`中的属性复制到当前描述的属性上，接着将这个 return 给`readClassDesc`的`validateDescriptor`

```
if (descriptor != null) {
    validateDescriptor(descriptor);
}
```

检查结束后，通过就 return 到`readOrdinaryObject`

```
private Object readOrdinaryObject(boolean unshared)
    throws IOException
{
    ...

    ObjectStreamClass desc = readClassDesc(false); 
    desc.checkDeserialize();

    ...
    Object obj;
    try {
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(
            desc.forClass().getName(),
            "unable to create instance").initCause(ex);
    }

    ...

    if (desc.isExternalizable()) {
        readExternalData((Externalizable) obj, desc);
    } else {
        readSerialData(obj, desc);
    }

    ...
}
```

看到对返回的 desc 进行反序列化检查`checkDeserialize()`

往下继续走

`desc.newInstance()`就是刚才的构造函数生成的组件

接着，当`desc`不是`Externalizable()`就会触发 else 的`readSerialData`

```
private void readSerialData(Object obj, ObjectStreamClass desc)
    throws IOException
{
    ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
    for (int i = 0; i < slots.length; i++) {
        ObjectStreamClass slotDesc = slots[i].desc;

        if (slots[i].hasData) {
            if (obj == null || handles.lookupException(passHandle) != null) {
                defaultReadFields(null, slotDesc); // skip field values
            } else if (slotDesc.hasReadObjectMethod()) {
                ThreadDeath t = null;
                boolean reset = false;
                SerialCallbackContext oldContext = curContext;
                if (oldContext != null)
                    oldContext.check();
                try {
                    curContext = new SerialCallbackContext(obj, slotDesc);

                    bin.setBlockDataMode(true);
                    slotDesc.invokeReadObject(obj, this);
                } catch (ClassNotFoundException ex) {
                    /*
                     * In most cases, the handle table has already
                     * propagated a CNFException to passHandle at this
                     * point; this mark call is included to address cases
                     * where the custom readObject method has cons'ed and
                     * thrown a new CNFException of its own.
                     */
                    handles.markException(passHandle, ex);
                } finally {
                    do {
                        try {
                            curContext.setUsed();
                            if (oldContext!= null)
                                oldContext.check();
                            curContext = oldContext;
                            reset = true;
                        } catch (ThreadDeath x) {
                            t = x;  // defer until reset is true
                        }
                    } while (!reset);
                    if (t != null)
                        throw t;
                }

                /*
                 * defaultDataEnd may have been set indirectly by custom
                 * readObject() method when calling defaultReadObject() or
                 * readFields(); clear it to restore normal read behavior.
                 */
                defaultDataEnd = false;
            } else {
                defaultReadFields(obj, slotDesc);
                }
    ...
        }
```

这里先进行判断，当我们没有重写`readObject`那么就调用`defaultReadFilelds`, 否则调用`slotDesc.invokeReadObject(obj,this)`

```
void invokeReadObject(Object obj, ObjectInputStream in)
    throws ClassNotFoundException, IOException,
           UnsupportedOperationException
{
    requireInitialized();
    if (readObjectMethod != null) {
        try {
            readObjectMethod.invoke(obj, new Object[]{ in });
        } catch (InvocationTargetException ex) {
            Throwable th = ex.getTargetException();
            if (th instanceof ClassNotFoundException) {
                throw (ClassNotFoundException) th;
            } else if (th instanceof IOException) {
                throw (IOException) th;
            } else {
                throwMiscException(th);
            }
        } catch (IllegalAccessException ex) {
            // should not occur, as access checks have been suppressed
            throw new InternalError(ex);
        }
    } else {
        throw new UnsupportedOperationException();
    }
}
```

读取我们重写的`readObject`

可以看到`readObjectMethod.invoke(obj, new Object[]{ in });`

这里的`readObjectMethod`就是我们自写的`readObject`方法

readObject 执行流程
---------------

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6w9Q1mURJcnMF7heoVUaWRGVDLxzar9NKkNGCbZCYVibyrtG8mRvjrWPAwC3PQgnAnbzWnfsO9autrw/640?wx_fmt=png)

Java 反序列化之 readObject 分析