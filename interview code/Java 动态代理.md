### Java 动态代理

1.在某些情况下，我们不希望或是不能直接访问对象 A，而是通过访问一个中介对象 B，由 B 去访问 A 达成目的，这种方式我们就称为代理。
这里对象 A 所属类我们称为委托类，也称为被代理类，对象 B 所属类称为代理类。

优点：

- 隐藏委托类的实现
- 解耦，不改变委托类代码情况下做一些额外处理，比如添加初始判断及其他公共操作

1.2 静态代理

代理类在程序运行前已经存在的代理方式称为静态代理。

静态代理中代理类和委托类也常常继承同一父类或实现同一接口。

```
class ClassA {
    public void operateMethod1() {};

    public void operateMethod2() {};

    public void operateMethod3() {};
}

public class ClassB {
    private ClassA a;

    public ClassB(ClassA a) {
        this.a = a;
    }

    public void operateMethod1() {
        a.operateMethod1();
    };

    public void operateMethod2() {
        a.operateMethod2();
    };

    // not export operateMethod3()
}
```

1.3 动态代理

代理类在程序运行前不存在、运行时由程序动态生成的代理方式称为动态代理。

Java 提供了动态代理的实现方式，可以在运行时刻动态生成代理类。这种代理方式的一大好处是可以方便对代理类的函数做统一或特殊处理，如记录所有函数执行时间、所有函数执行前添加验证判断、对某个特殊函数进行特殊操作，而不用像静态代理方式那样需要修改每个函数。

2.动态代理

#### 实现动态代理包括三步：

(1). 新建委托类；
(2). 实现`InvocationHandler`接口，这是负责连接代理类和委托类的中间类必须实现的接口；
(3). 通过`Proxy`类新建代理类对象。

假如现在我们想统计某个类所有函数的执行时间，传统的方式是在类的每个函数前打点统计，动态代理方式如下：

2.1 新建委托类

```
public interface Operate {

    public void operateMethod1();

    public void operateMethod2();

    public void operateMethod3();
}

public class OperateImpl implements Operate {

    @Override
    public void operateMethod1() {
        System.out.println("Invoke operateMethod1");
        sleep(110);
    }

    @Override
    public void operateMethod2() {
        System.out.println("Invoke operateMethod2");
        sleep(120);
    }

    @Override
    public void operateMethod3() {
        System.out.println("Invoke operateMethod3");
        sleep(130);
    }

    private static void sleep(long millSeconds) {
        try {
            Thread.sleep(millSeconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

`Operate`是一个接口，定了了一些函数，我们要统计这些函数的执行时间。
`OperateImpl`是委托类，实现`Operate`接口。每个函数简单输出字符串，并等待一段时间。
动态代理要求委托类必须实现了某个接口，比如这里委托类`OperateImpl`实现了`Operate`，

2.2 实现InvocationHandler接口

```
public class TimingInvocationHandler implements InvocationHandler {

    private Object target;

    public TimingInvocationHandler() {}

    public TimingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long start = System.currentTimeMillis();
        Object obj = method.invoke(target, args);
        System.out.println(method.getName() + " cost time is:" + (System.currentTimeMillis() - start));
        return obj;
    }
}
```

`target`属性表示委托类对象。

`InvocationHandler`是负责连接代理类和委托类的中间类必须实现的接口。其中只有一个

```
public Object invoke(Object proxy, Method method, Object[] args)

```

函数需要去实现，参数：
`proxy`表示下面`2.3 通过 Proxy.newProxyInstance() 生成的代理类对象`。
`method`表示代理对象被调用的函数。
`args`表示代理对象被调用的函数的参数。

调用代理对象的每个函数实际最终都是调用了`InvocationHandler`的`invoke`函数。这里我们在`invoke`实现中添加了开始结束计时，其中还调用了委托类对象`target`的相应函数，这样便完成了统计执行时间的需求。
`invoke`函数中我们也可以通过对`method`做一些判断，从而对某些函数特殊处理。

2.3 通过Proxy类静态函数生成代理对象

```
public class Main {
    public static void main(String[] args) {
        // create proxy instance
        TimingInvocationHandler timingInvocationHandler = new TimingInvocationHandler(new OperateImpl());
        Operate operate = (Operate)(Proxy.newProxyInstance(Operate.class.getClassLoader(), new Class[] {Operate.class},
                timingInvocationHandler));

        // call method of proxy instance
        operate.operateMethod1();
        System.out.println();
        operate.operateMethod2();
        System.out.println();
        operate.operateMethod3();
    }
}
```

然后通过`Proxy.newProxyInstance(…)`函数新建了一个代理对象，实际代理类就是在这时候动态生成的。我们调用该代理对象的函数就会调用到`timingInvocationHandler`的`invoke`函数(是不是有点类似静态代理)，而`invoke`函数实现中调用委托类对象`new OperateImpl()`相应的 method(是不是有点类似静态代理)。

```
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

`loader`表示类加载器
`interfaces`表示委托类的接口，生成代理类时需要实现这些接口
`h`是`InvocationHandler`实现类对象，负责连接代理类和委托类的中间类

我们可以这样理解，如上的动态代理实现实际是双层的静态代理，开发者提供了委托类 B，程序动态生成了代理类 A。开发者还需要提供一个实现了`InvocationHandler`的子类 C，子类 C 连接代理类 A 和委托类 B，它是代理类 A 的委托类，委托类 B 的代理类。用户直接调用代理类 A 的对象，A 将调用转发给委托类 C，委托类 C 再将调用转发给它的委托类 B。

3.动态代理的原理

3.1  newProxyInstance()

```
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
    if (h == null) {
        throw new NullPointerException();
    }

    /*
     * Look up or generate the designated proxy class.
     */
    Class cl = getProxyClass(loader, interfaces);

    /*
     * Invoke its constructor with the designated invocation handler.
     */
    try {
        Constructor cons = cl.getConstructor(constructorParams);
        return (Object) cons.newInstance(new Object[] { h });
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString());
    } catch (IllegalAccessException e) {
        throw new InternalError(e.toString());
    } catch (InstantiationException e) {
        throw new InternalError(e.toString());
    } catch (InvocationTargetException e) {
        throw new InternalError(e.toString());
    }
}
```

从中可以看出它先调用`getProxyClass(loader, interfaces)`得到动态代理类，然后将`InvocationHandler`作为代理类构造函数入参新建代理类对象。

3.2 getProxyClass()

```
/**
 * 得到代理类，不存在则动态生成
 * @param loader 代理类所属 ClassLoader
 * @param interfaces 代理类需要实现的接口
 * @return
 */
public static Class<?> getProxyClass(ClassLoader loader,
                                     Class<?>... interfaces)
    throws IllegalArgumentException
{
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }
    // 代理类类对象
    Class proxyClass = null;

    /* collect interface names to use as key for proxy class cache */
    String[] interfaceNames = new String[interfaces.length];

    Set interfaceSet = new HashSet();       // for detecting duplicates

    /**
     * 入参 interfaces 检验，包含三部分
     * （1）是否在入参指定的 ClassLoader 内
     * （2）是否是 Interface
     * （3）interfaces 中是否有重复
     */
    for (int i = 0; i < interfaces.length; i++) {
        String interfaceName = interfaces[i].getName();
        Class interfaceClass = null;
        try {
            interfaceClass = Class.forName(interfaceName, false, loader);
        } catch (ClassNotFoundException e) {
        }
        if (interfaceClass != interfaces[i]) {
            throw new IllegalArgumentException(
                interfaces[i] + " is not visible from class loader");
        }

        if (!interfaceClass.isInterface()) {
            throw new IllegalArgumentException(
                interfaceClass.getName() + " is not an interface");
        }

        if (interfaceSet.contains(interfaceClass)) {
            throw new IllegalArgumentException(
                "repeated interface: " + interfaceClass.getName());
        }
        interfaceSet.add(interfaceClass);

        interfaceNames[i] = interfaceName;
    }

    // 以接口名对应的 List 作为缓存的 key
    Object key = Arrays.asList(interfaceNames);

    /*
     * loaderToCache 是个双层的 Map
     * 第一层 key 为 ClassLoader，第二层 key 为 上面的 List，value 为代理类的弱引用
     */
    Map cache;
    synchronized (loaderToCache) {
        cache = (Map) loaderToCache.get(loader);
        if (cache == null) {
            cache = new HashMap();
            loaderToCache.put(loader, cache);
        }
    }

    /*
     * 以上面的接口名对应的 List 为 key 查找代理类，如果结果为：
     * (1) 弱引用，表示代理类已经在缓存中
     * (2) pendingGenerationMarker 对象，表示代理类正在生成中，等待生成完成通知。
     * (3) null 表示不在缓存中且没有开始生成，添加标记到缓存中，继续生成代理类
     */
    synchronized (cache) {
        do {
            Object value = cache.get(key);
            if (value instanceof Reference) {
                proxyClass = (Class) ((Reference) value).get();
            }
            if (proxyClass != null) {
                // proxy class already generated: return it
                return proxyClass;
            } else if (value == pendingGenerationMarker) {
                // proxy class being generated: wait for it
                try {
                    cache.wait();
                } catch (InterruptedException e) {
                }
                continue;
            } else {
                cache.put(key, pendingGenerationMarker);
                break;
            }
        } while (true);
    }

    try {
        String proxyPkg = null;     // package to define proxy class in

        /*
         * 如果 interfaces 中存在非 public 的接口，则所有非 public 接口必须在同一包下面，后续生成的代理类也会在该包下面
         */
        for (int i = 0; i < interfaces.length; i++) {
            int flags = interfaces[i].getModifiers();
            if (!Modifier.isPublic(flags)) {
                String name = interfaces[i].getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {     // if no non-public proxy interfaces,
            proxyPkg = "";          // use the unnamed package
        }

        {
            // 得到代理类的类名，jdk 1.6 版本中缺少对这个生成类已经存在的处理。
            long num;
            synchronized (nextUniqueNumberLock) {
                num = nextUniqueNumber++;
            }
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            // 动态生成代理类的字节码
            // 最终调用 sun.misc.ProxyGenerator.generateClassFile() 得到代理类相关信息写入 DataOutputStream 实现
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces);
            try {
                // native 层实现，虚拟机加载代理类并返回其类对象
                proxyClass = defineClass0(loader, proxyName,
                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                throw new IllegalArgumentException(e.toString());
            }
        }
        // add to set of all generated proxy classes, for isProxyClass
        proxyClasses.put(proxyClass, null);

    } finally {
        // 代理类生成成功则保存到缓存，否则从缓存中删除，然后通知等待的调用
        synchronized (cache) {
            if (proxyClass != null) {
                cache.put(key, new WeakReference(proxyClass));
            } else {
                cache.remove(key);
            }
            cache.notifyAll();
        }
    }
    return proxyClass;
}

```

**函数主要包括三部分：**

- 入参 interfaces 检验，包含是否在入参指定的 ClassLoader 内、是否是 Interface、interfaces 中是否有重复
- 以接口名对应的 List 为 key 查找代理类，如果结果为：
  - 弱引用，表示代理类已经在缓存中；
  - pendingGenerationMarker 对象，表示代理类正在生成中，等待生成完成返回；
  - null 表示不在缓存中且没有开始生成，添加标记到缓存中，继续生成代理类。
- 如果代理类不存在调用`ProxyGenerator.generateProxyClass(…)`生成代理类并存入缓存，通知在等待的缓存。

**函数中几个注意的地方：**

- 代理类的缓存 key 为接口名对应的 List，接口顺序不同表示不同的 key 即不同的代理类。
- 如果 interfaces 中存在非 public 的接口，则所有非 public 接口必须在同一包下面，后续生成的代理类也会在该包下面。
- 代理类如果在 ClassLoader 中已经存在的情况没有做处理。
- 可以开启 System Properties 的`sun.misc.ProxyGenerator.saveGeneratedFiles`开关，保存动态类到目的地址。

Java 1.7 的实现略有不同，通过`getProxyClass0(…)`函数实现，实现中调用代理类的缓存，判断代理类在缓存中是否已经存在，存在直接返回，不存在则调用`proxyClassCache`的`valueFactory`属性进行动态生成，`valueFactory`的`apply`函数与上面的`getProxyClass(…)`函数逻辑类似。



